
https://www.3dgep.com/forward-plus/


```
cd thirdparty\boost_1_70_0
bootstrap.bat
b2 toolset=msvc-14.2 --build-type=complete --abbreviate-paths architecture=x86 address-model=64 install -j4 stage

build-sln-open.bat
```
