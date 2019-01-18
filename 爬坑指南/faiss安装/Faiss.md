### faiss 编译安装

#### 安装依赖

```
yum install llvm-toolset-7-libomp-devel -y
yum install openblas-devel.x86_64 -y
```

#### 安装swig3.0

```
tar -zxvf swig-3&
cd swig-3*
./configure
make
make install
```

#### 安装faiss

这步可以省略，仅需要将附件中的faiss文件夹拷贝到*/opt/rh/rh-python36/root/lib64/python3.6/site-packages*

然后安装预编译版的faiss

```
pip3 install faiss-prebuilt   
```