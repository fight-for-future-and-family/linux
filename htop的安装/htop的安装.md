##  htop的安装
 
 
#### 一、下载需要的文件
wget http://ftp.gnu.org/pub/gnu/ncurses/ncurses-5.9.tar.gz
wget http://downloads.sourceforge.net/project/htop/htop/1.0.1/htop-1.0.1.tar.gz

---

#### 二、编译安装

tar xvfz ncurses-5.9.tar.gz
cd ncurses-5.9
./configure
make
make install

tar xzvf htop-1.0.1.tar.gz
cd htop-1.0.1
./configure
make
make install