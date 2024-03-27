# 001 - [[override]]

```c++
struct Foo { virtual void DoThing(); };

struct Bar : Foo {
	virtual void DoThing() override;
}; 
```