## 1. 背景知识    

cublas 中默认的矩阵存储继承了 fortran 的存储格式 -- 列主序（column-major)，和我们在c/c++中使用的行主序（row-major)有所不同，列主序的时候每一列内部的元素是连续的，行主序的每一行内部的元素是不连续的。  


C-style 的行主序是这样的：

cublas 看到的这个矩阵，内存布局是这样的：

![image](https://github.com/user-attachments/assets/4d6f0b35-07db-4b59-baaa-23c145b35670)  
*The memory layout of a matrix in C-style indexing* 

但是在 cublas 看来，内存布局是这样的：  

![image](https://github.com/user-attachments/assets/bc3ac203-f991-4455-b52f-edba12262dd1)
*The memory layout of a matrix in cublas indexing*  

## 2. 不同 layout 的矩阵在 cublas 中怎么配置  

第一部分为背景知识，以下的所有讲解都以 cublas 的背景知识为基础。  cublasSgemm的函数原型为：
```c++
cublasStatus_t cublasSgemm(cublasHandle_t handle,
                           cublasOperation_t transa, cublasOperation_t transb,
                           int m, int n, int k,
                           const float           *alpha,
                           const float           *A, int lda,
                           const float           *B, int ldb,
                           const float           *beta,
                           float           *C, int ldc)
```
**transa, transb**: 是否需要对输入的矩阵做转置操作，比如 A（3*5）， B（3 *5）,A和B无法直接相乘，必须先B做转置，A*B^T才是有意义的。  
**A，B**： 指向两个矩阵的起始位置。  
**lda, ldb, ldc**: cublas视角下，同一行相邻列在内存中的距离。和 Opencv中 CV::Mat 的 stride 代表的意义是一样的。  
**C**： 输出的矩阵地址。  

如果是不熟悉 cublas 或者 cuda 相关库，一开始配置这些参数很容易陷入混乱，尤其是对哪些需要进行转置的矩阵。下面举几个例子来说明一下，cublasSgemm的参数该如何设置。

### 2.1. 两个 C-style row-major 矩阵相乘  

矩阵A的尺寸为 3 * 5， 矩阵B的尺寸为 5 * 2，相乘的结果C的尺寸为 3 * 2

a00, a01, a02, a03, a04
a10, a11, a12, a13, a14
a20, a21, a22, a23, a24
**C-style视角下的矩阵 A， 3行5列**

b00, b01
b10, b11
b20, b21
b30, b31
b40, b41
**C-style 视角下的矩阵 B，5行，2列**

以上两个 row-major 矩阵在内存中分别存储在连续的空间中，当通过 cudaMemcpy 拷贝到 GPU 的时候，cublas视角下的内存布局是：  

a00, a10, a20  
a01, a11, a21  
a02, a12, a22  
a03, a13, a23  
a04, a14, a24  
*cublas 视角下的矩阵A， 5行3列*

b00, b10, b20, b30, b40  
b01, b11, b21, b31, b41  
*cublas 视角下的矩阵B，2行5列*

C-style 视角下的 AB = C，在 cublas 视角下是： A^T * B^T = C, 根据矩阵运算的一些法则，又等同于：B*A=C^T

c00, c01, c02
c10, c11, c12

所以: 
transa = N, transb = N, 
lda = 5, ldb = 2
ldc = 3 



## 参考文献  

1.  [CuBlas matrix multiplication with C-style arrays](https://web.archive.org/web/20230325211919/https://peterwittek.com/2013/06/cublas-matrix-c-style/)  
2.

