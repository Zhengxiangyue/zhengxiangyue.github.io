```bash
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

```bash
cd opencv
<cmake> -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local -DOPENCV_EXTRA_MODULES_PATH=<opencv_contrib>/modules ..
```

My Cmake path is 

> /Applications/CLion.app/Contents/bin/cmake/mac/bin/cmake

My opencv_contrib path is 

> /Users/zhengxiangyue/Projects/opencv_contrib

Because we need to `#include "opencv2/face.hpp"`,  so opencv_contrib is needed

Then

```
make -j7 # runs 7 jobs in parallel
```
<span style="color:red">This should be red</span>

