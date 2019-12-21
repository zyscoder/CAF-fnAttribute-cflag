# add-fcaf-keep-cxxctors-and-cafcxxctor-function-attribute.patch 

- 实现功能：
    1. 给clang增加一个 '-fcaf-keep-cxxctors' 的cflag，
       实现了将所有的c++ constructor全都emit到ir，即使其没有被当前模块的任何函数调用过。
       并防止后续的pass将保留下来的c++ constructors优化掉或内联掉，
       使其保持单个函数的形式，知道被编译成binary。
    2. 在clang和llvm上，实现了一个自定义的function attribute: cafcxxctor,
       这个attribute是用来标记所有c++ constructor函数的，
       被上述第1点功能处理过的程序所产生的ir，所有的c++ constructors都会带上cafcxxctor这个属性，
       便于后续将所有函数构造器筛选出来，进行后续的程序分析等。

- 实现原理：
    1. '-fcaf-keep-cxxctors' cflag的实现原理：在ast emit到ir的过程中，会对每一个函数进行判断其是否需要emit i.e. bool CodeGenModule::MustBeEmitted(const ValueDecl *Global).
       在这个过程中，判断每一个函数是否是CXXConstructorDecl类型，
       如果是，为其加上OptimizeNoneAttr, NoInlineAttr, WeakImportAttr, CafCxxCtorAttr等属性。
       其中最重要的是OptimizeNoneAttr属性，它会告诉后面的pass，当前函数不需要优化。
    2. CafCxxCtorAttr的实现原理：跟着OptimizeNoneAttr的实现过程，一步步实现自定义的attr即可。

# add-cafapi-function-attribute.patch

- 实现功能：给clang和llvm增加一个‘cafapi’的function attribute，可用以在源码标识目标函数，方便在ir pass中进行识别和处理。

- 实现环境：
    - llvm7.1.0: 本patch适用于llvm7.1.0版本，不保证其他版本的适用性

- patch使用方法：
   - git apply /path/to/patchname
