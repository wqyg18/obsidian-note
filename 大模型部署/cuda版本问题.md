`nvidia-smi`显示的只是已经安装的驱动的版本, 同时其显示的`CUDA Version: 1x.y`并非是实际安装的cuda的版本, 而是当前安装的驱动支持的最高的cuda 的版本

查看当前安装的版本需要 `nvcc --version`

另外`nvcc`是**编译器**。把 CUDA C++ 代码编译成显卡能懂的指令