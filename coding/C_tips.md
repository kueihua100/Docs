# C/C++ tips
### How to handle error of unused parameter?
    Compile error:
      error: unused parameter 'arg' [-Werror,-Wunused-parameter]
      
    1) casting
        #define UNUSED(x) (void)(x)
        void func(int arg) {
            UNUSED(arg);
            ...
        }
        
    2) mark parameters (C++)
        void func(int /* arg */) {
            ...
        }
        
    3) std::ignore (C++11)
        #include <tuple>  //need to include this header
        void func(int arg) {
            std::ignore = arg;
        }
        
    4) function attribute unused (GCC/G++)
        void func(int arg __attribute__((unused))) {
            ...
        }

    5) [[maybe_unused]] (C++17)
        void func([[maybe_unused]] int arg) {
            ...
        }
