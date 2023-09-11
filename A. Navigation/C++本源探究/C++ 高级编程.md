
### 静态多态

```c++
#include <iostream>
#include <typeinfo>

template <typename Derived> struct Animal {
	void bark() {
		std::cout << typeid(*this).name() << std::endl;
		static_cast<Derived&>(*this).barkImpl(); // or next line
		// static_cast<Derived*>(this)->barkImpl()
	}
};

struct Cat : public Animal<Cat> {
	void barkImpl() {
		std::cout << typeid(*this).name() << std::endl;
		std::cout << "MiaoWing!" << std::endl;
	}
};

struct Dog : public Animal<Dog> {
	void barkImpl() {
		std::cout << typeid(*this).name() << std::endl;
		std::cout << "DogWing!" << std::endl;
	}
};

template<typename T> void play(Animal<T>& animal) {
	animal.bark();
} 

int main() {
	Cat cat; play(cat);
	Dog dog; play(dog);
    return 0;
}
```