


optional容器 
nullopt
has_value
value

```c++

std::optional<float> mysqrt(float x){
	if(x >= 0.f){
		return std::sqrt(x);
	}else{
		return std::nullopt;
	}
}


auto ret = mysqrt(3.f);
if(ret.has_value()) cout<<ret.value()<<endl;
else balabala

```


value_or(没有的话选用的值)