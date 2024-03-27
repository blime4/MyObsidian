# 004 - Variable Template

Before c++14 we had Function Template and Class Template, now we have Varivable Template

```c++
template<typename T>
T pi = T(3.1415926535897932384626433L);

int main(){
	cout.precision(std::numeric_limits<long double>::max_digits10);
	std::cout << pi<int> <<endl;
	std::cout << pi<float> <<endl;
	std::cout << pi<double> <<endl;
	std::cout << pi<long double> <<endl;
	return 0;
}
```