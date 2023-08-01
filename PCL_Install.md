# vmware ubuntu 20.04 pcl install
#### 参考链接
> * [PCL入门教程](https://www.yuque.com/huangzhongqing/pcl/aghhcg "PCL入门教程")
> * [vmware_ubuntu_20.4 pcl安装](https://blog.csdn.net/weixin_44708246/article/details/130439855?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-4-130439855-blog-120827256.235^v38^pc_relevant_anti_vip_base&spm=1001.2101.3001.4242.3&utm_relevant_index=7 "vmware ubuntu 20.04 pcl安装")
> * [安装pcl的详细步骤与走过的坑](https://www.cnblogs.com/linzzz98/p/13746949.html "安装pcl的详细步骤与走过的坑")
> * [ubuntu20.4安装pcl](https://blog.csdn.net/qq_41092406/article/details/117930972 "ubuntu20.4安装pcl")

***

* 安装PCL各种依赖
~~~
# 可以将此段内容保存为 install.sh, 使用'sh install.sh'命令自动执行所有指令
sudo apt-get update
sudo apt-get install git build-essential linux-libc-dev
sudo apt-get install cmake cmake-gui
sudo apt-get install libusb-1.0-0-dev libusb-dev libudev-dev
sudo apt-get install mpi-default-dev openmpi-bin openmpi-common
sudo apt-get install libflann1.9 libflann-dev  # ubuntu20.4对应1.9
sudo apt-get install libeigen3-dev
sudo apt-get install libboost-all-dev
sudo apt-get install libqhull* libgtest-dev
sudo apt-get install freeglut3-dev pkg-config
sudo apt-get install libxmu-dev libxi-dev
sudo apt-get install mono-complete
sudo apt-get install libopenni-dev
sudo apt-get install libopenni2-dev
~~~

***

* 安装VTK
  * 下载地址：[VTK-7.1.1下载地址(an earlier release)](https://vtk.org/download/#earlier "VTK下载")
  * 安装VTK各种依赖
    ~~~
    # 首先安装VTK的依赖：X11，OpenGL；cmake和cmake-gui在安装pcl依赖的时候安装过了的话可以跳过
    # X11
    sudo apt-get install libx11-dev libxext-dev libxtst-dev libxrender-dev libxmu-dev libxmuu-dev
    # OpenGL
    sudo apt-get install build-essential libgl1-mesa-dev libglu1-mesa-dev
    # cmake && cmake-gui
    sudo apt-get install cmake cmake-gui
    ~~~
  * 将下载好的文件解压
    ~~~
    tar -zxvf xxx.tar.gz -c 指定路径
    ~~~
  * 在解压好的文件中新建build目录，在build目录中打开终端输入  'cmake-gui', 然后出现下方窗口
  ![figure1](https://img-blog.csdnimg.cn/fa52f5ed2ba04d148c8b4e3e0541dd54.png "figure1")
    * 'where is the source code'选择VTK文件夹的路径
    * 'where to build the binaries'选择刚刚创建的build文件夹路径
    * 点击【Configure】,等待出现'configuring done'
    * 勾选'VTK_Group_Qt'选项，其中VTK_QT_VERSION选择5，点击【configure】，等待出现'configuring done'
    * 最后点击【Generate】,等待出现'Generating done'，完成
 * 在build目录下重新打开一个终端输入
   ~~~
   make -j8
   sudo make install
   ~~~

***

* 安装PCL
  ~~~
  sudo apt install libpcl-dev --fix-missing
  sudo apt install pcl-tools
  ~~~

***

* 代码测试
  * 新建一个文件夹用于代码测试'pcl_test',在该文件夹中新建build文件夹，pcl_test.cpp以及CMakeLists.txt
  * 其中pcl_test.cpp中写入以下内容：
    ~~~
    #include <iostream>
    #include <pcl/common/common_headers.h>
    #include <pcl/io/pcd_io.h>
    #include <pcl/visualization/pcl_visualizer.h>
    #include <pcl/visualization/cloud_viewer.h>
    #include <pcl/console/parse.h>
     
     
    int main(int argc, char **argv) {
        std::cout << "Test PCL !!!" << std::endl;
        
        pcl::PointCloud<pcl::PointXYZRGB>::Ptr point_cloud_ptr (new pcl::PointCloud<pcl::PointXYZRGB>);
        uint8_t r(255), g(15), b(15);
        for (float z(-1.0); z <= 1.0; z += 0.05)
        {
          for (float angle(0.0); angle <= 360.0; angle += 5.0)
          {
    	pcl::PointXYZRGB point;
    	point.x = 0.5 * cosf (pcl::deg2rad(angle));
    	point.y = sinf (pcl::deg2rad(angle));
    	point.z = z;
    	uint32_t rgb = (static_cast<uint32_t>(r) << 16 |
    		static_cast<uint32_t>(g) << 8 | static_cast<uint32_t>(b));
    	point.rgb = *reinterpret_cast<float*>(&rgb);
    	point_cloud_ptr->points.push_back (point);
          }
          if (z < 0.0)
          {
    	r -= 12;
    	g += 12;
          }
          else
          {
    	g -= 12;
    	b += 12;
          }
        }
        point_cloud_ptr->width = (int) point_cloud_ptr->points.size ();
        point_cloud_ptr->height = 1;
        
        pcl::visualization::CloudViewer viewer ("test");
        viewer.showCloud(point_cloud_ptr);
        while (!viewer.wasStopped()){ };
        return 0;
    }
    ~~~
  * CMakeLists.txt中写入以下内容：
    ~~~
    cmake_minimum_required(VERSION 2.6)
    project(pcl_test)
    
    find_package(PCL 1.2 REQUIRED)
    
    include_directories(${PCL_INCLUDE_DIRS})
    link_directories(${PCL_LIBRARY_DIRS})
    add_definitions(${PCL_DEFINITIONS})
    
    add_executable(pcl_test pcl_test.cpp)
    
    target_link_libraries (pcl_test ${PCL_LIBRARIES})
    
    install(TARGETS pcl_test RUNTIME DESTINATION bin)
    ~~~
  * 在命令行执行以下命令
    ~~~
    cd build
    cmake ..
    make
    ./pcl_test
    ~~~
  * 运行成功将得到一个3D模型
#### 参考链接
> * [PCL入门教程](https://www.yuque.com/huangzhongqing/pcl/aghhcg "PCL入门教程")
> * [vmware_ubuntu_20.4 pcl安装](https://blog.csdn.net/weixin_44708246/article/details/130439855?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-4-130439855-blog-120827256.235^v38^pc_relevant_anti_vip_base&spm=1001.2101.3001.4242.3&utm_relevant_index=7 "vmware ubuntu 20.04 pcl安装")
> * [安装pcl的详细步骤与走过的坑](https://www.cnblogs.com/linzzz98/p/13746949.html "安装pcl的详细步骤与走过的坑")
> * [ubuntu20.4安装pcl](https://blog.csdn.net/qq_41092406/article/details/117930972 "ubuntu20.4安装pcl")
