### TPU-MLIR 架构

![[Pasted image 20230722143046.png]]

### Defining a Dialect

```c++
/// This is the definition of the Toy dialect. A dialect inherits from
/// mlir::Dialect and registers custom attributes, operations, and types. It can
/// also override virtual methods to change some general behavior, which will be
/// demonstrated in later chapters of the tutorial.
class ToyDialect : public mlir::Dialect {
public:
  explicit ToyDialect(mlir::MLIRContext *ctx);

  /// Provide a utility accessor to the dialect namespace.
  static llvm::StringRef getDialectNamespace() { return "toy"; }

  /// An initializer called from the constructor of ToyDialect that is used to
  /// register attributes, operations, types, and more within the Toy dialect.
  void initialize();
};
```

**注意:** 与该`dialect`相关的 `attributes, operations, types` 需要通过`initialize`函数内部注册
```c++
void TopDialect::initialize() {
  addAttributes<
#define GET_ATTRDEF_LIST
#include "tpu_mlir/Dialect/Top/IR/TopAttr.cpp.inc"
      >();
  addOperations<
#define GET_OP_LIST
#include "tpu_mlir/Dialect/Top/IR/TopOps.cpp.inc"
      >();
}
```

### Defining a Dialect via [tablegen](https://llvm.org/docs/TableGen/ProgRef.html)

```tablegen
// Provide a definition of the 'toy' dialect in the ODS framework so that we
// can define our operations.
def Toy_Dialect : Dialect {
  // The namespace of our dialect, this corresponds 1-1 with the string we
  // provided in `ToyDialect::getDialectNamespace`.
  let name = "toy";

  // A short one-line summary of our dialect.
  let summary = "A high-level dialect for analyzing and optimizing the "
                "Toy language";

  // A much longer description of our dialect.
  let description = [{
    The Toy language is a tensor-based language that allows you to define
    functions, perform some math computation, and print results. This dialect
    provides a representation of the language that is amenable to analysis and
    optimization.
  }];

  // The C++ namespace that the dialect class definition resides in.
  let cppNamespace = "toy";
}
```

After the dialect has been defined, it can now be loaded into an MLIRContext:

```c++
// context.loadDialect<ToyDialect>(); or via DialectRegistry
void registerAllDialects(mlir::DialectRegistry &registry) {
  registry
      .insert<mlir::tosa::TosaDialect, mlir::func::FuncDialect, top::TopDialect,
              tpu::TpuDialect, mlir::quant::QuantizationDialect>();
}
```
By default, an `MLIRContext` only loads the [Builtin Dialect](https://mlir.llvm.org/docs/Dialects/Builtin/), which provides a few core IR components, meaning that other dialects, such as our `Toy` dialect, must be explicitly loaded.


![[Pasted image 20230722143553.png]]
![[Pasted image 20230722143817.png]]

### MLIR示例
![[Pasted image 20230722143951.png]]
![[Pasted image 20230722144032.png]]
**ModuleOp 和FuncOp 都可以视作Operation。**
![[Pasted image 20230722144530.png]]

#### Defining  Operations

Shown here is the general form of an operation:

- A name for the operation.
- A list of SSA operand values.
- A list of [attributes](https://mlir.llvm.org/docs/LangRef/#attributes).
- A list of [types](https://mlir.llvm.org/docs/LangRef/#type-system) for result values.
- A [source location](https://mlir.llvm.org/docs/Diagnostics/#source-locations) for debugging purposes.
- A list of successors [blocks](https://mlir.llvm.org/docs/LangRef/#blocks) (for branches, mostly).
- A list of [regions](https://mlir.llvm.org/docs/LangRef/#regions) (for structural operations like functions).

```c++
%t_tensor = "toy.transpose"(%tensor) {inplace = true} : (tensor<2x3xf64>) -> tensor<3x2xf64> loc("example/file/path":12:1)
```

**An operation class inherits from the [CRTP](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern) `mlir::Op` class which also takes some optional [_traits_](https://mlir.llvm.org/docs/Tutorials/Traits.md) to customize its behavior.** `Traits` are a mechanism with which we can inject additional behavior into an Operation, such as additional accessors, verification, and more.
```c++
class ConstantOp : public mlir::Op<
                     /// `mlir::Op` is a CRTP class, meaning that we provide the
                     /// derived class as a template parameter.
                     ConstantOp,
                     /// The ConstantOp takes zero input operands.
                     mlir::OpTrait::ZeroOperands,
                     /// The ConstantOp returns a single result.
                     mlir::OpTrait::OneResult,
                     /// We also provide a utility `getType` accessor that
                     /// returns the TensorType of the single result.
                     mlir::OpTraits::OneTypedResult<TensorType>::Impl> {

 public:
  /// Inherit the constructors from the base Op class.
  using Op::Op;

  /// Provide the unique name for this operation. MLIR will use this to register
  /// the operation and uniquely identify it throughout the system. The name
  /// provided here must be prefixed by the parent dialect namespace followed
  /// by a `.`.
  static llvm::StringRef getOperationName() { return "toy.constant"; }

  /// Return the value of the constant by fetching it from the attribute.
  mlir::DenseElementsAttr getValue();

  /// Operations may provide additional verification beyond what the attached
  /// traits provide.  Here we will ensure that the specific invariants of the
  /// constant operation are upheld, for example the result type must be
  /// of TensorType and matches the type of the constant `value`.
  LogicalResult verifyInvariants();

  /// Provide an interface to build this operation from a set of input values.
  /// This interface is used by the `builder` classes to allow for easily
  /// generating instances of this operation:
  ///   mlir::OpBuilder::create<ConstantOp>(...)
  /// This method populates the given `state` that MLIR uses to create
  /// operations. This state is a collection of all of the discrete elements
  /// that an operation may contain.
  /// Build a constant with the given return type and `value` attribute.
  static void build(mlir::OpBuilder &builder, mlir::OperationState &state,
                    mlir::Type result, mlir::DenseElementsAttr value);
  /// Build a constant and reuse the type from the given 'value'.
  static void build(mlir::OpBuilder &builder, mlir::OperationState &state,
                    mlir::DenseElementsAttr value);
  /// Build a constant by broadcasting the given 'value'.
  static void build(mlir::OpBuilder &builder, mlir::OperationState &state,
                    double value);
};
```

and we can register this operation in the `ToyDialect` initializer:

```c++
void ToyDialect::initialize() {
  addOperations<ConstantOp>();
}
```

operation 是一个基本执行单元，它的构建是为了能够在特定dialect下对源码或上一层IR进行表达，并通过operation对其进行各种变换。

#### Op vs Operation: Using MLIR Operations

+ In MLIR, there are two main classes related to operations: `Operation` and `Op`.
+ The `Operation` class is used to generically model all operations.
	+ It is ‘opaque’, in the sense that it does not describe the properties of particular operations or types of operations.
	+  the `Operation` class provides a general API into an operation instance.
+ Each specific type of operation is represented by an `Op` derived class
+ `Op` derived classes act as smart pointer wrapper around a `Operation*`, provide operation-specific accessor methods, and type-safe properties of operations.
+ A side effect of this design is that we always pass around `Op` derived classes “by-value”, instead of by reference or pointer(_passing by value_ is a common idiom in MLIR and applies similarly to attributes, types, etc).
+ Given a generic `Operation*` instance, we can always get a specific `Op` instance using LLVM’s casting infrastructure:

```c++
void processConstantOp(mlir::Operation *operation) {
  ConstantOp op = llvm::dyn_cast<ConstantOp>(operation);

  // This operation is not an instance of `ConstantOp`.
  if (!op)
    return;

  // Get the internal operation instance wrapped by the smart pointer.
  mlir::Operation *internalOperation = op.getOperation();
  assert(internalOperation == operation &&
         "these operation instances are the same");
}
```

![[Pasted image 20230722151206.png]]

#### Using the [Operation Definition Specification](https://mlir.llvm.org/docs/DefiningDialects/Operations/) (ODS) Framework

```c++
// Base class for toy dialect operations. This operation inherits from the base
// `Op` class in OpBase.td, and provides:
//   * The parent dialect of the operation.
//   * The mnemonic for the operation, or the name without the dialect prefix.
//   * A list of traits for the operation.
class Toy_Op<string mnemonic, list<Trait> traits = []> :
    Op<Toy_Dialect, mnemonic, traits>;
```

We define a toy operation by inheriting from our base ‘Toy_Op’ class above. Here we provide the mnemonic and a list of traits for the operation. The [mnemonic](https://mlir.llvm.org/docs/DefiningDialects/Operations/#operation-name) here matches the one given in `ConstantOp::getOperationName` without the dialect prefix; `toy.`. Missing here from our C++ definition are the `ZeroOperands` and `OneResult` traits; these will be automatically inferred based upon the `arguments` and `results` fields we define later.

```c++
def ConstantOp : Toy_Op<"constant"> {
  // Provide a summary and description for this operation. This can be used to
  // auto-generate documentation of the operations within our dialect.
  let summary = "constant operation";
  let description = [{
    Constant operation turns a literal into an SSA value. The data is attached
    to the operation as an attribute. For example:

      %0 = "toy.constant"()
         { value = dense<[[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]]> : tensor<2x3xf64> }
        : () -> tensor<2x3xf64>
  }];

  // The constant operation takes an attribute as the only input.
  // `F64ElementsAttr` corresponds to a 64-bit floating-point ElementsAttr.
  let arguments = (ins F64ElementsAttr:$value);

  // The generic call operation returns a single value of TensorType.
  // F64Tensor corresponds to a 64-bit floating-point TensorType.
  let results = (outs F64Tensor);

  // Add additional verification logic to the constant operation. Setting this bit
  // to `1` will generate a `::mlir::LogicalResult verify()` declaration on the
  // operation class that is called after ODS constructs have been verified, for
  // example the types of arguments and results. We implement additional verification
  // in the definition of this `verify` method in the C++ source file. 
  let hasVerifier = 1;
}
```
#### Verifying Operation Semantics

To add additional verification logic, an operation can override the [`verifier`](https://mlir.llvm.org/docs/DefiningDialects/Operations/#custom-verifier-code) field. The `verifier` field allows for defining a C++ code blob that will be run as part of `ConstantOp::verify`. This blob can assume that all of the other invariants of the operation have already been verified:

#### Attaching `build` Methods

ODS can generate some simple build methods automatically, and in this case it will generate our first build method for us. For the rest, we define the [`builders`](https://mlir.llvm.org/docs/DefiningDialects/Operations/#custom-builder-methods) field. This field takes a list of `OpBuilder` objects that take a string corresponding to a list of C++ parameters, as well as an optional code block that can be used to specify the implementation inline.

```c++
def ConstantOp : Toy_Op<"constant"> {
  ...

  // Add custom build methods for the constant operation. These methods populate
  // the `state` that MLIR uses to create operations, i.e. these are used when
  // using `builder.create<ConstantOp>(...)`.
  let builders = [
    // Build a constant with a given constant tensor value.
    OpBuilder<(ins "DenseElementsAttr":$value), [{
      // Call into an autogenerated `build` method.
      build(builder, result, value.getType(), value);
    }]>,

    // Build a constant with a given constant floating-point value. This builder
    // creates a declaration for `ConstantOp::build` with the given parameters.
    OpBuilder<(ins "double":$value)>
  ];
}
```

Value 有两个派生类 BlockArgument 和 OpResult
![[Pasted image 20230722151331.png]]
**通用表示的是获得基类指针**，**特定表示获得派生类指针**。
**mlir::Type可以自行定义。**
可以通过hasRank接口判断某Type是否为Rank。
![[Pasted image 20230722151630.png]]
**Atrribute可以自行定义**
![[Pasted image 20230722151758.png]]
![[Pasted image 20230722152635.png]]
**Op定义方式**
![[Pasted image 20230722153209.png]]
![[Pasted image 20230722153427.png]]
![[Pasted image 20230722153610.png]]
![[Pasted image 20230722154132.png]]
![[Pasted image 20230722155433.png]]
![[Pasted image 20230722155733.png]]
![[Pasted image 20230722161039.png]]
### MLIR- Dialect Conversion
![[Pasted image 20230722164430.png]]
![[Pasted image 20230722164544.png]]
![[Pasted image 20230722165036.png]]
![[Pasted image 20230722165251.png]]
![[Pasted image 20230722165350.png]]

### Lowering in TPU-MLIR

![[Pasted image 20230722163654.png]]
![[Pasted image 20230722164018.png]]
![[Pasted image 20230722175019.png]]
### Pattern Rewriting
![[Pasted image 20230722203605.png]]
![[Pasted image 20230722205652.png]]
![[Pasted image 20230722210116.png]]
![[Pasted image 20230722210252.png]]
+ 定义好pattern之后，我们需要把所有会用到的pattern输送给driver来应用这些patterns。
+ driver是通过RewritePatternSet形式来接收所有patterns, 所以我们需要先将创建好的patterns添加到其中。
![[Pasted image 20230722211552.png]]

### 如何在MLIR里面写Pass

在[【从零开始学深度学习编译器】十一，初识MLIR](https://mp.weixin.qq.com/s?__biz=MzA4MjY4NTk0NQ==&mid=2247499292&idx=1&sn=149110d0d6fcec34856125ec01e4d38b&scene=21#wechat_redirect) 和 [【从零开始学深度学习编译器】十二，MLIR Toy Tutorials学习笔记一](https://mp.weixin.qq.com/s?__biz=MzA4MjY4NTk0NQ==&mid=2247499384&idx=1&sn=1e2d3ca811c2047d7d9e6c13e63bd91c&scene=21#wechat_redirect) 这两篇文章中，我们已经初步了解了MLIR为何物，并且讲到了Toy语言从源文件生成MLIR的具体过程，以及在这个过程中MLIR中的MLIRGen，Dialect，Operation以及TableGen这几个MLIR的核心组成部分以及它们是如何相互作用的。

这篇笔记将基于Toy Tutorials总结MLIR中的表达式变形是如何实现的。

#### MLIR中的表达式变形(如何写Pass)

在Chapter2中我们已经生成了初级的合法MLIR表达式，但MLIR表达式一般还可以被进一步处理和简化，可以类比于TVM的Pass对Relay IR的优化。这里我们来看看要对初级的MLIR表达式进行变形是如何做的？在MLIR中是基于表达式匹配和重写来完成MLIR表达式变形的。这个教程中分别介绍使用C++模板匹配和重写以及基于DRR框架（`https://mlir.llvm.org/docs/DeclarativeRewrites/`）来定义表达式重写规则，然后使用ODS框架来自动生成代码。

**1. 使用C++模式匹配和重写的方法优化转置（Transpose）操作**

这里的目标是要消除两个具有相互抵消效果的转置序列：`transpose(transpose(X)) -> X`，即对同一个输入进行连续的Transpose操作肯定存在冗余的操作。该操作对应的源码如下（在`mlir/test/Examples/Toy/Ch3/transpose_transpose.toy`中）：

```c++
def transpose_transpose(x) {  
  return transpose(transpose(x));  
}
```

如果不使用任何优化Pass，我们看下这个Toy源程序生成的MLIR表达式是什么样子的，使用下面的命令产生MLIR：`./toyc-ch3 ../../mlir/test/Examples/Toy/Ch3/transpose_transpose.toy -emit=mlir`。

```c++
func @transpose_transpose(%arg0: tensor<*xf64>) -> tensor<*xf64> {  
    %0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64>  
    %1 = toy.transpose(%0 : tensor<*xf64>) to tensor<*xf64>  
    toy.return %1 : tensor<*xf64>  
  }
```

可以看到生成的MLIR表达式中对`x`进行了两次真正的transpose操作，并且返回了两次transpose之后的Tensor。但实际上这两次transpose是不必要的，因为输出的结果其实就是传入的`x`。所以为了优化这种情况，我们先使用C++方式来写出表达式匹配和重写的代码（在`mlir/examples/toy/Ch3/mlir/ToyCombine.cpp`中）：

```c++
/// This is an example of a c++ rewrite pattern for the TransposeOp. It  
/// optimizes the following scenario: transpose(transpose(x)) -> x  
struct SimplifyRedundantTranspose : public mlir::OpRewritePattern<TransposeOp> {  
  /// We register this pattern to match every toy.transpose in the IR.  
  /// The "benefit" is used by the framework to order the patterns and process  
  /// them in order of profitability.  
  SimplifyRedundantTranspose(mlir::MLIRContext *context)  
      : OpRewritePattern<TransposeOp>(context, /*benefit=*/1) {}  
  
  /// This method attempts to match a pattern and rewrite it. The rewriter  
  /// argument is the orchestrator of the sequence of rewrites. The pattern is  
  /// expected to interact with it to perform any changes to the IR from here.  
  mlir::LogicalResult  
  matchAndRewrite(TransposeOp op,  
                  mlir::PatternRewriter &rewriter) const override {  
    // Look through the input of the current transpose.  
    mlir::Value transposeInput = op.getOperand();  
    TransposeOp transposeInputOp = transposeInput.getDefiningOp<TransposeOp>();  
  
    // Input defined by another transpose? If not, no match.  
    if (!transposeInputOp)  
      return failure();  
  
    // Otherwise, we have a redundant transpose. Use the rewriter.  
    rewriter.replaceOp(op, {transposeInputOp.getOperand()});  
    return success();  
  }  
};
```

可以看到在`matchAndRewrite`函数中，首先获取当前操作的操作数，然后判断当前位置的操作数对应的操作是否为转置，如果是就将表达式重写为内层转置操作的操作数，不然就不需要进行优化，保持现状。

接下来，需要在归范化框架（Canonicalization Framework）中注册刚刚创建的匹配重写模式，使得框架可以调用它。对于Canonicalization 更多的介绍请看`https://mlir.llvm.org/docs/Canonicalization/`，注册的代码如下（代码仍在：`mlir/examples/toy/Ch3/mlir/ToyCombine.cpp`）：

```c++
/// Register our patterns as "canonicalization" patterns on the TransposeOp so  
/// that they can be picked up by the Canonicalization framework.  
void TransposeOp::getCanonicalizationPatterns(RewritePatternSet &results,  
                                              MLIRContext *context) {  
  results.add<SimplifyRedundantTranspose>(context);  
}
```

在我们将表达式重写规则添加到了规范化框架后，我们还需要修改一下定义Operator的`td`文件，启用规范化框架，同时在定义Operator添加一个“无副作用的”(`NoSideEffect`)新特征，现在Transpose操作的定义如下：

```c++
def TransposeOp : Toy_Op<"transpose", [NoSideEffect]> {  
  let summary = "transpose operation";  
  
  let arguments = (ins F64Tensor:$input);  
  let results = (outs F64Tensor);  
  
  let assemblyFormat = [{  
    `(` $input `:` type($input) `)` attr-dict `to` type(results)  
  }];  
  
  // Enable registering canonicalization patterns with this operation.  
  let hasCanonicalizer = 1;  
  
  // Allow building a TransposeOp with from the input operand.  
  let builders = [  
    OpBuilder<(ins "Value":$input)>  
  ];  
  
  // Invoke a static verify method to verify this transpose operation.  
  let verifier = [{ return ::verify(*this); }];  
}
```

最后，我们需要在主程序中将基于规范化框架的优化添加到运行流程里，这部分代码在`mlir/examples/toy/Ch3/toyc.cpp`中的`dumpMLIR`函数里面。如下图的黄色涂鸦部分

![[Pasted image 20230725155014.png]]

可以看到优化后的MLIR表达式已经去掉了transpose操作了，达到了优化效果。

**2. 使用 DRR 优化张量变形（Reshape）操作**

MLIR还提供了一种表达式重写的方法，是基于DDR规则的方式来自动生成表达式匹配和重写函数，代码生成的部分仍然基于ODS框架实现。DRR（Declarative, Rule-based Pattern-match and Rewrite）：声明性、基于规则的模式匹配和重写方法。它是一种基于 DAG 的声明性重写器，提供基于表格的模式匹配和重写规则的句法。

这里以消除MLIR表达式中冗余的张量reshape操作为例，对应的Toy源文件如下（在`mlir/test/Examples/Toy/Ch3/trivial_reshape.toy`中）：

```c++
def main() {  
  var a<2,1> = [1, 2];  
  var b<2,1> = a;  
  var c<2,1> = b;  
  print(c);  
}
```

使用下面的命令先产生对应的MLIR表达式看看：`./toyc-ch3 ../../mlir/test/Examples/Toy/Ch3/trivial_reshape.toy -emit=mlir`

```c++
module  {  
  func @main() {  
    %0 = toy.constant dense<[1.000000e+00, 2.000000e+00]> : tensor<2xf64>  
    %1 = toy.reshape(%0 : tensor<2xf64>) to tensor<2x1xf64>  
    %2 = toy.reshape(%1 : tensor<2x1xf64>) to tensor<2x1xf64>  
    %3 = toy.reshape(%2 : tensor<2x1xf64>) to tensor<2x1xf64>  
    toy.print %3 : tensor<2x1xf64>  
    toy.return  
  }  
}
```

很明显`a`，`b`，`c`的shape和值都是一样的，这些reshape操作是多余的。下面我们要基于DDR框架来定义表达式匹配和重写规则。这里要分几种情况考虑（这里的代码实现都在`mlir/examples/toy/Ch3/mlir/ToyCombine.td`）。

- 解决`Reshape(Reshape(x)) = Reshape(x)`产生的冗余代码。

```c++
// Reshape(x) = x, where input and output shapes are identical  
def TypesAreIdentical : Constraint<CPred<"$0.getType() == $1.getType()">>;  
def RedundantReshapeOptPattern : Pat<  
  (ReshapeOp:$res $arg), (replaceWithValue $arg),  
  [(TypesAreIdentical $res, $arg)]>;
```

即当`0.getType()`与`1.getType()`相同时即为冗余，使用操作数`$arg`代替。

Some optimizations may require additional transformations on instruction arguments. This is achieved using NativeCodeCall, which allows for more complex transformations either by calling into a C++ helper function or by using inline C++. An example of such an optimization is FoldConstantReshape, where we optimize Reshape of a constant value by reshaping the constant in place and eliminating the reshape operation.

```c++
def ReshapeConstant : NativeCodeCall<"$0.reshape(($1.getType()).cast<ShapedType>())">;
def FoldConstantReshapeOptPattern : Pat<
  (ReshapeOp:$res (ConstantOp $arg)),
  (ConstantOp (ReshapeConstant $arg, $res))>;
```

接下来我们就可以使用 ODS 框架和定义好的 `ToyCombine.td` 文件，自动化生成代码文件 `ToyCombine.inc`。使用下面的命令：

```shell
$   cd llvm-project/build  
$   ./bin/mlir-tblgen --gen-rewriters ${mlir_src_root}/examples/toy/Ch3/mlir/ToyCombine.td -I ${mlir_src_root}/include/
```

当然构建工程的时候也可以将这个生成过程配置在cmakelists.txt中：`mlir/examples/toy/Ch3/CMakeLists.txt`。如下：

```shell
set(LLVM_TARGET_DEFINITIONS mlir/ToyCombine.td)  
mlir_tablegen(ToyCombine.inc -gen-rewriters)  
add_public_tablegen_target(ToyCh3CombineIncGen)
```

最后，我们可以执行`./toyc-ch3 ../../mlir/test/Examples/Toy/Ch3/trivial_reshape.toy -emit=mlir -opt`生成经过这些Pass优化的MLIR表达式：

```c++
module  {  
  func @main() {  
    %0 = toy.constant dense<[[1.000000e+00], [2.000000e+00]]> : tensor<2x1xf64>  
    toy.print %0 : tensor<2x1xf64>  
    toy.return  
  }  
}
```

**3. 实现泛化的表达式转化**

在上边我们学到了如何在MLIR里面实现表达式重写，但上面也有一个非常明显的问题：我们为Toy语言实现的Pass在其它的Dialect抽象中没办法重用，因为这里只是针对Toy语言的一些Operation的特化操作，如果为每种Dialect实现每种转化会导致大量重复代码。所以，这一节以两个例子为例讲解如何在MLIR中实现泛化的表达式。

本文使用下面的例子进行介绍(在`mlir/test/Examples/Toy/Ch5/codegen.toy`)：

```c++
def multiply_transpose(a, b) {  
  return transpose(a) * transpose(b);  
}  
  
def main() {  
  var a<2, 3> = [[1, 2, 3], [4, 5, 6]];  
  var b<2, 3> = [1, 2, 3, 4, 5, 6];  
  var c = multiply_transpose(a, b);  
  var d = multiply_transpose(b, a);  
  print(d);  
}
```

我们先看一下它对应的MLIR表达式`./toyc-ch4 ../../mlir/test/Examples/Toy/Ch4/codegen.toy -emit=mlir`：

```c++
module  {  
  func private @multiply_transpose(%arg0: tensor<*xf64>, %arg1: tensor<*xf64>) -> tensor<*xf64> {  
    %0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64>  
    %1 = toy.transpose(%arg1 : tensor<*xf64>) to tensor<*xf64>  
    %2 = toy.mul %0, %1 : tensor<*xf64>  
    toy.return %2 : tensor<*xf64>  
  }  
  func @main() {  
    %0 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64>  
    %1 = toy.reshape(%0 : tensor<2x3xf64>) to tensor<2x3xf64>  
    %2 = toy.constant dense<[1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00]> : tensor<6xf64>  
    %3 = toy.reshape(%2 : tensor<6xf64>) to tensor<2x3xf64>  
    %4 = toy.generic_call @multiply_transpose(%1, %3) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>  
    %5 = toy.generic_call @multiply_transpose(%3, %1) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>  
    toy.print %5 : tensor<*xf64>  
    toy.return  
  }  
}
```

**3.1 内联Pass**

观察上面的代码我们可以发现`multiply_transpose`这种小函数被频繁调用，这个时候函数调用本身的开销就不容忽视。所以这里定义一个内联Pass希望把`multiply_transpose`这个函数变成内联函数以提高运行效率。
**<center>第一步</center>

MLIR提供了一个处理内联的通用接口`DialectInlinerInterface` ，它包含一组Dialect可以重写的虚拟钩子，我们要基于这个类为Toy Operation定义内联的接口和表达式重写规则。代码实现在：`mlir/examples/toy/Ch5/mlir/Dialect.cpp`：

```c++
// This class defines the interface for handling inlining with Toy operations.
/// We simplify inherit from the base interface class and override
/// the necessary methods.
struct ToyInlinerInterface : public DialectInlinerInterface {
  using DialectInlinerInterface::DialectInlinerInterface;

  /// This hook checks to see if the given callable operation is legal to inline
  /// into the given call. For Toy this hook can simply return true, as the Toy
  /// Call operation is always inlinable.
  bool isLegalToInline(Operation *call, Operation *callable,
                       bool wouldBeCloned) const final {
    return true;
  }

  /// This hook checks to see if the given operation is legal to inline into the
  /// given region. For Toy this hook can simply return true, as all Toy
  /// operations are inlinable.
  bool isLegalToInline(Operation *, Region *, bool,
                       IRMapping &) const final {
    return true;
  }

  /// This hook cheks if the given 'src' region can be inlined into the 'dest'
  /// region. The regions here are the bodies of the callable functions. For
  /// Toy, any function can be inlined, so we simply return true.
  bool isLegalToInline(Region *dest, Region *src, bool wouldBeCloned,
                       IRMapping &valueMapping) const final {
    return true;
  }

  /// This hook is called when a terminator operation has been inlined. The only
  /// terminator that we have in the Toy dialect is the return
  /// operation(toy.return). We handle the return by replacing the values
  /// previously returned by the call operation with the operands of the
  /// return.
  void handleTerminator(Operation *op,
                        ArrayRef<Value> valuesToRepl) const final {
    // Only "toy.return" needs to be handled here.
    auto returnOp = cast<ReturnOp>(op);

    // Replace the values directly with the return operands.
    assert(returnOp.getNumOperands() == valuesToRepl.size());
    for (const auto &it : llvm::enumerate(returnOp.getOperands()))
      valuesToRepl[it.index()].replaceAllUsesWith(it.value());
  }
};
```

这部分代码为Toy Operation定义了内联的接口和表达式变形的规则，两个`isLegalToInline`重载函数是两个钩子。第一个钩子用来检查给定的可调用操作`callable`内联到给定调用`call`中是否合法，检查是否可以内联。第二个钩子用来检查给定的操作是否合法地内联到给定的区域。`handleTerminator`函数只是处理`toy.return`，将返回操作的操作数`it.index()`直接用返回值`it.value()`代替（这里没太懂QAQ）。

Besides, the inliner will only discard private-visible unused function definitions. We also have to set the visibility of functions (except the main function) in the MLIR generator.

> 这段话所说的意思是，在 MLIR 的内联优化中，只有私有可见的未使用函数定义（Private-visible unused function definitions）才会被丢弃，即不会被包含在生成的目标代码中。因此，为了确保未使用的函数在目标代码中被正确地丢弃，我们还需要在 

> MLIR 生成器中设置函数的可见性（visibility）。
> 在 MLIR 中，函数的可见性可以通过在函数的属性（attribute）中设置 `visibility` 字段来指定。例如，可以将一个函数的可见性设置为 `private`，表示该函数只在模块内可见，而不会被导出到目标代码中。如果一个函数未被使用，且其可见性被设置为 `private`，则该函数的定义会被丢弃，不会出现在生成的目标代码中。

> 需要注意的是，这里所说的函数可见性和 MLIR 中的 `private` 修饰符并不是一回事。`private` 修饰符是在 MLIR 代码中使用的一个关键字，用于指定某个变量或函数只在当前模块内可见，而不会被导出到其他模块中。而函数的可见性是在生成目标代码时使用的一个概念，用于指定哪些函数应该被包含在目标代码中，哪些函数应该被丢弃。

```c++
/// Emit a new function and add it to the MLIR module.
mlir::toy::FuncOp mlirGen(FunctionAST &funcAST) {
  ...
  // If this function isn't main, then set the visibility to private.
  if (funcAST.getProto()->getName() != "main")
    function.setPrivate();

  return function;
}
```

<center>第二步</center>

接着，需要在Toy Dialect的定义中添加上面的表达式变形规则，位置在`mlir/examples/toy/Ch5/mlir/Dialect.cpp`。

```c++
/// Dialect initialization, the instance will be owned by the context. This is  
/// the point of registration of types and operations for the dialect.  
void ToyDialect::initialize() {  
  addOperations<  
#define GET_OP_LIST  
#include "toy/Ops.cpp.inc"  
      >();  
  addInterfaces<ToyInlinerInterface>();  
}
```


<center>第三步</center>

再接着，我们需要让内联器`inliner`知道IR中`toy.generic_call`表示的是调用一个函数。MLIR提供了一个Operation接口`CallOpInterface`可以将某个Operation标记为调用。添加上述操作需要在Toy Dialect的定义(`mlir/examples/toy/Ch5/include/toy/Ops.td`)文件中加入`include "mlir/Interfaces/CallInterfaces.td"`这行代码。

然后在Dialect定义部分添加一个新的Operation，代码如下所示：

```c++
def FuncOp : Toy_Op<"func", [
    DeclareOpInterfaceMethods<CallableOpInterface>, FunctionOpInterface,
    IsolatedFromAbove
  ]> {
  let summary = "user defined function operation";
  let description = [{
    The "toy.func" operation represents a user defined function. These are
    callable SSA-region operations that contain toy computations.

    Example:

    ``mlir
    toy.func @main() {
      %0 = toy.constant dense<5.500000e+00> : tensor<f64>
      %1 = toy.reshape(%0 : tensor<f64>) to tensor<2x2xf64>
      toy.print %1 : tensor<2x2xf64>
      toy.return
    }
    ``
  }];

  let arguments = (ins
    SymbolNameAttr:$sym_name,
    TypeAttrOf<FunctionType>:$function_type,
    OptionalAttr<DictArrayAttr>:$arg_attrs,
    OptionalAttr<DictArrayAttr>:$res_attrs
  );
  let regions = (region AnyRegion:$body);

  let builders = [OpBuilder<(ins
    "StringRef":$name, "FunctionType":$type,
    CArg<"ArrayRef<NamedAttribute>", "{}">:$attrs)
  >];
  let extraClassDeclaration = [{
    //===------------------------------------------------------------------===//
    // FunctionOpInterface Methods
    //===------------------------------------------------------------------===//

    /// Returns the argument types of this function.
    ArrayRef<Type> getArgumentTypes() { return getFunctionType().getInputs(); }

    /// Returns the result types of this function.
    ArrayRef<Type> getResultTypes() { return getFunctionType().getResults(); }
  }];
  let hasCustomAssemblyFormat = 1;
  let skipDefaultBuilders = 1;
}


def GenericCallOp : Toy_Op<"generic_call",
    [DeclareOpInterfaceMethods<CallOpInterface>]> {
  let summary = "generic call operation";
  let description = [{
    Generic calls represent calls to a user defined function that needs to
    be specialized for the shape of its arguments. The callee name is attached
    as a symbol reference via an attribute. The arguments list must match the
    arguments expected by the callee. For example:

    ```mlir
     %4 = toy.generic_call @my_func(%1, %3)
           : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>
    ` This is only valid if a function named "my_func" exists and takes two
    arguments.
    ``
}];

  // The generic call operation takes a symbol reference attribute as the
  // callee, and inputs for the call.
  let arguments = (ins FlatSymbolRefAttr:$callee, Variadic<F64Tensor>:$inputs);

  // The generic call operation returns a single value of TensorType.
  let results = (outs F64Tensor);

  // Specialize assembly printing and parsing using a declarative format.
  let assemblyFormat = [{
    $callee `(` $inputs `)` attr-dict `:` functional-type($inputs, results)
  }];

  // Add custom build methods for the generic call operation.
  let builders = [
    OpBuilder<(ins "StringRef":$callee, "ArrayRef<Value>":$arguments)>
  ];
}
```

在`mlir/examples/toy/Ch5/mlir/Dialect.cpp`中实现

```c++
/// Returns the region on the function operation that is callable.
Region *FuncOp::getCallableRegion() { return &getBody(); }

/// Returns the results types that the callable region produces when
/// executed.
ArrayRef<Type> FuncOp::getCallableResults() { return getType().getResults(); }

/// Returns the argument attributes for all callable region arguments or
/// null if there are none.
ArrayAttr FuncOp::getCallableArgAttrs() {
  return getArgAttrs().value_or(nullptr);
}

/// Returns the result attributes for all callable region results or
/// null if there are none.
ArrayAttr FuncOp::getCallableResAttrs() {
  return getResAttrs().value_or(nullptr);
}

// ....

/// Return the callee of the generic call operation, this is required by the
/// call interface.
CallInterfaceCallable GenericCallOp::getCallableForCallee() {
  return getAttrOfType<SymbolRefAttr>("callee");
}

/// Set the callee for the generic call operation, this is required by the call
/// interface.
void GenericCallOp::setCalleeFromCallable(CallInterfaceCallable callee) {
  (*this)->setAttr("callee", callee.get<SymbolRefAttr>());
}

/// Get the argument operands to the called function, this is required by the
/// call interface.
Operation::operand_range GenericCallOp::getArgOperands() { return inputs(); }
```

Now that the inliner has been informed about the Toy dialect, we can add the inliner pass to the pass manager for Toy:

```c++
pm.addPass(mlir::createInlinerPass());
```

Now let’s look at a working example:

```c++
toy.func @multiply_transpose(%arg0: tensor<*xf64>, %arg1: tensor<*xf64>) -> tensor<*xf64> {
  %0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64>
  %1 = toy.transpose(%arg1 : tensor<*xf64>) to tensor<*xf64>
  %2 = toy.mul %0, %1 : tensor<*xf64>
  toy.return %2 : tensor<*xf64>
}
toy.func @main() {
  %0 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64>
  %1 = toy.reshape(%0 : tensor<2x3xf64>) to tensor<2x3xf64>
  %2 = toy.constant dense<[1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00]> : tensor<6xf64>
  %3 = toy.reshape(%2 : tensor<6xf64>) to tensor<2x3xf64>
  %4 = toy.generic_call @multiply_transpose(%1, %3) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>
  %5 = toy.generic_call @multiply_transpose(%3, %1) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64>
  toy.print %5 : tensor<*xf64>
  toy.return
}
```

We have two calls to multiply_transpose that we would like to inline into main, but if we look at the output nothing has changed. We are missing one last subtle piece: there is a hidden type conversion on the edge of the call. If we look at the above, the operands to the generic_call are of type `tensor<2x3xf64>`, while the inputs to the function expect `tensor<*xf64>`. To resolve this difference, the inliner expects an explicit cast operation to be inserted. For this, we need to add a new operation to the Toy dialect, `ToyCastOp`(toy.cast), to represent casts between two different shapes.

```c++
def CastOp : Toy_Op<"cast", [
    DeclareOpInterfaceMethods<CastOpInterface>,
    Pure,
    SameOperandsAndResultShape]
  > {
  let summary = "shape cast operation";
  let description = [{
    The "cast" operation converts a tensor from one type to an equivalent type
    without changing any data elements. The source and destination types
    must both be tensor types with the same element type. If both are ranked,
    then shape is required to match. The operation is invalid if converting
    to a mismatching constant dimension.
  }];

  let arguments = (ins F64Tensor:$input);
  let results = (outs F64Tensor:$output);
  let assemblyFormat = "$input attr-dict `:` type($input) `to` type($output)";
}
```

Note that the definition of this cast operation adds a `CastOpInterface` to the traits list. This interface provides several utilities for cast-like operation, such as folding identity casts and verification. We hook into this interface by providing a definition for the `areCastCompatible` method:

```c++
/// Returns true if the given set of input and result types are compatible with
/// this cast operation. This is required by the `CastOpInterface` to verify
/// this operation and provide other additional utilities.
bool CastOp::areCastCompatible(TypeRange inputs, TypeRange outputs) {
  if (inputs.size() != 1 || outputs.size() != 1)
    return false;
  // The inputs must be Tensors with the same element type.
  TensorType input = inputs.front().dyn_cast<TensorType>();
  TensorType output = outputs.front().dyn_cast<TensorType>();
  if (!input || !output || input.getElementType() != output.getElementType())
    return false;
  // The shape is required to match if both types are ranked.
  return !input.hasRank() || !output.hasRank() || input == output;
}
```

With a proper cast operation, we can now override the necessary hook on the ToyInlinerInterface to insert it for us when necessary:

```c++
struct ToyInlinerInterface : public DialectInlinerInterface {
  ...

  /// Attempts to materialize a conversion for a type mismatch between a call
  /// from this dialect, and a callable region. This method should generate an
  /// operation that takes 'input' as the only operand, and produces a single
  /// result of 'resultType'. If a conversion can not be generated, nullptr
  /// should be returned.
  Operation *materializeCallConversion(OpBuilder &builder, Value input,
                                       Type resultType,
                                       Location conversionLoc) const final {
    return builder.create<CastOp>(conversionLoc, resultType, input);
  }
};
```

If we run the working example through the pipeline again, we get the expected:

```c++
toy.func @main() {
  %0 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64>
  %1 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64>
  %2 = toy.cast %1 : tensor<2x3xf64> to tensor<*xf64>
  %3 = toy.cast %0 : tensor<2x3xf64> to tensor<*xf64>
  %4 = toy.transpose(%2 : tensor<*xf64>) to tensor<*xf64>
  %5 = toy.transpose(%3 : tensor<*xf64>) to tensor<*xf64>
  %6 = toy.mul %4, %5 : tensor<*xf64>
  toy.print %6 : tensor<*xf64>
  toy.return
}
```

**3.2 Shape推断 Pass**

<center>第一步：使用ODS框架定义Shape推断Operation接口</center>

上面内联Pass实现了将确定类型的Tensor转换成了泛化类型的Tensor，进而使得内联操作得以完成。然后接下来，我们需要根据形状确定的Tensor来推导那些泛化Tensor的形状。这里需要利用ODS框架来生成自定义的Operation接口来推导泛化Tensor的形状。整个Shape推断的过程也会和inline一样抽象成一个Pass作用在MLIR表达式上。

```c++
def ShapeInferenceOpInterface : OpInterface<"ShapeInference"> {  
  let description = [{  
    Interface to access a registered method to infer the return types for an  
    operation that can be used during type inference.  
  }];  
  
  let methods = [  
    InterfaceMethod<"Infer and set the output shape for the current operation.",  
                    "void", "inferShapes">  
  ];  
}
```

<center>第二步：将特征添加到必要的 Toy Operation定义中</center>

以Toy语言的Mul Operation为例，实现在`mlir/examples/toy/Ch5/include/toy/Ops.td`：

```c++
def MulOp : Toy_Op<"mul",  
    [NoSideEffect, DeclareOpInterfaceMethods<ShapeInferenceOpInterface>]> {  
  let summary = "element-wise multiplication operation";  
  let description = [{  
    The "mul" operation performs element-wise multiplication between two  
    tensors. The shapes of the tensor operands are expected to match.  
  }];  
  
  let arguments = (ins F64Tensor:$lhs, F64Tensor:$rhs);  
  let results = (outs F64Tensor);  
  
  // Specify a parser and printer method.  
  let parser = [{ return ::parseBinaryOp(parser, result); }];  
  let printer = [{ return ::printBinaryOp(p, *this); }];  
  
  // Allow building a MulOp with from the two input operands.  
  let builders = [  
    OpBuilder<(ins "Value":$lhs, "Value":$rhs)>  
  ];  
}
```

<center>第三步：定义对应Operation的形状推导函数</center>
需要进行形状推导的每个Operation，都需要定义对应的`inferShapes()`函数，比如Mul Operation，结果的形状就是输入的形状（因为是elementwise操作）。代码实现在`mlir/examples/toy/Ch5/mlir/Dialect.cpp`：

```c++
/// Infer the output shape of the MulOp, this is required by the shape inference
/// interface.
void MulOp::inferShapes() { getResult().setType(getLhs().getType()); }
```


<center>第四步：实现形状推导Pass</center>

这一步是介绍形状推导Pass的具体实现，前面几步是这一步的前置条件。这一步定义一个形状推导Pass类来实现Shape推断算法，并会基于这个Pass类来创建一个Shape推断的Pass。代码实现在`mlir/examples/toy/Ch5/mlir/ShapeInferencePass.cpp`。

```c++
class ShapeInferencePass  
    : public mlir::PassWrapper<ShapeInferencePass, FunctionPass> {  
public:  
  void runOnFunction() override {  
    auto f = getFunction();  
  
    // Populate the worklist with the operations that need shape inference:  
    // these are operations that return a dynamic shape.  
    llvm::SmallPtrSet<mlir::Operation *, 16> opWorklist;  
    f.walk([&](mlir::Operation *op) {  
      if (returnsDynamicShape(op))  
        opWorklist.insert(op);  
    });  
  
    // Iterate on the operations in the worklist until all operations have been  
    // inferred or no change happened (fix point).  
    while (!opWorklist.empty()) {  
      // Find the next operation ready for inference, that is an operation  
      // with all operands already resolved (non-generic).  
      auto nextop = llvm::find_if(opWorklist, allOperandsInferred);  
      if (nextop == opWorklist.end())  
        break;  
  
      Operation *op = *nextop;  
      opWorklist.erase(op);  
  
      // Ask the operation to infer its output shapes.  
      LLVM_DEBUG(llvm::dbgs() << "Inferring shape for: " << *op << "\n");  
      if (auto shapeOp = dyn_cast<ShapeInference>(op)) {  
        shapeOp.inferShapes();  
      } else {  
        op->emitError("unable to infer shape of operation without shape "  
                      "inference interface");  
        return signalPassFailure();  
      }  
    }  
  
    // If the operation worklist isn't empty, this indicates a failure.  
    if (!opWorklist.empty()) {  
      f.emitError("Shape inference failed, ")  
          << opWorklist.size() << " operations couldn't be inferred\n";  
      signalPassFailure();  
    }  
  }  
  
  /// A utility method that returns if the given operation has all of its  
  /// operands inferred.  
  static bool allOperandsInferred(Operation *op) {  
    return llvm::all_of(op->getOperandTypes(), [](Type operandType) {  
      return operandType.isa<RankedTensorType>();  
    });  
  }  
  
  /// A utility method that returns if the given operation has a dynamically  
  /// shaped result.  
  static bool returnsDynamicShape(Operation *op) {  
    return llvm::any_of(op->getResultTypes(), [](Type resultType) {  
      return !resultType.isa<RankedTensorType>();  
    });  
  }  
};  
} // end anonymous namespace  
  
/// Create a Shape Inference pass.  
std::unique_ptr<mlir::Pass> mlir::toy::createShapeInferencePass() {  
  return std::make_unique<ShapeInferencePass>();  
}
```

<center>第五步：把形状推导Pass加到优化pipline</center>

While at it, let’s also create a helper method for instantiating the pass:

```c++
std::unique_ptr<mlir::Pass> mlir::toy::createShapeInferencePass() {
  return std::make_unique<ShapeInferencePass>();
}
```

When processing an operation like described, we query if it registered the `ShapeInference` interface, using this code snippet:

We can then add our pass to the pass manager:

```c++
  pm.addPass(mlir::createShapeInferencePass());
```
![[Pasted image 20230725213625.png]]