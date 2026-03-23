[CUDA Toolkit Archive \| NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)
选择需要的版本

然后执行安装, 只安装`nvcc`
```powershell
sh cuda_12.8.0_570.86.10_linux.run \
  --toolkit \
  --silent \
  --override \
  --installpath=$HOME/cuda-12.8
```

之后添加环境变量到`.bashrc`
```
export CUDA_HOME=$HOME/cuda-12.8
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```