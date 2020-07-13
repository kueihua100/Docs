# Tips for libatsc3 
## Compile code
* Install packages: 

  sudo apt-get install libncursesw5-dev
  
  sudo apt-get install libpcap-dev 
  
  sudo apt-get install ffmpeg 
  
* Install cmake (>3.10 verson)

  wget http://www.cmake.org/files/v3.12/cmake-3.12.1.tar.gz 
  
  tar -xvzf cmake-3.12.1.tar.gz 
  
  cd cmake-3.12.1/ 
  
  ./configure 
  
  make 
  
  sudo make install   /* will install at usr/local/bin */ 
  
  add /usr/local/bin to your PATH 

* run libatsc3/src/build_linux.sh to build libmicrohttpd and libap4

  at my environment I marked below code at build_linux.sh 
  
  #sudo ln -s /usr/include/locale.h /usr/include/xlocale.h 
  
  The reason to mark it due to both header files are exist at my platform and need the step to enter sudo PASSWD.

* After libmicrohttpd and libap4 are esixted, to build the libatsc3 is only below steps:
  
  cd src; make all
