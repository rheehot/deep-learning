# xwMOOC 딥러닝
`r Sys.Date()`  




## 1. 텐서플로우 사전준비 {#tensorflow-setup}

텐서플로우를 R에서 사용하고자 할 경우, 파이썬 설치만으로 문제가 해결되지 않는 경우가 있다.
만약 CPU 버젼만 사용한다면 그다지 문제가 될 것은 없다. 하지만,
하드웨어로 GPU를 사용하고자 할 경우 NVidia 그래픽카드를 사용할 수 있도록 툴체인을 사전에 설치해야된다.

- CUDA® Toolkit 8.0: 자세한 내용 [CUDA Installation Guide for Microsoft Windows](http://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/) 참조
- CUDA® Toolkit 8.0과 연관된 NVIDIA 드라이버: CUDA® Toolkit 8.0을 설치하면 자동으로 맞춰준다.
- cuDNN v6 or v6.1: [NVIDIA cuDNN - GPU Accelerated Deep Learning](https://developer.nvidia.com/cudnn) 참조
- CUDA와 GPU 그래픽카드 호환성 검사: [CUDA GPUs](https://developer.nvidia.com/cuda-gpus) 참조.

`CUDA® Toolkit 8.0`을 설치하면 시스템 경로에 자동으로 추가된다. 만약 환경변수 시스템 경로에 추가되지 않았다면 수작업으로 넣어둔다.
`cuDNN v6 or v6.1`은 zip 파일로 담겨졌기 때문에 압축을 풀고 나서 환경변수 시스템 경로에 추가한다.

윈도우에서 GPU 그래픽카드 하드웨어를 딥러닝 텐서플로우를 활용하고자 할 경우 환경설정을 맞추기가 쉽지 않다.
많은 사람들이 어려움을 겪고 있어 이를 파이썬 스크립트로 정리한 것이 있어 다음 스크립트를 활용하여 
빠른 시간내에 설정을 해본다. `tensorflow_self_check.py` 파이썬 스크립트를 다운로드 받아 파이썬에서 돌려보면 
텐서플로우 설정에서 부족한 부분을 검사하여 준다.

- [TensorFlow on Windows self-check](https://gist.github.com/mrry/ee5dbcfdd045fa48a27d56664411d41c)


~~~{.r}
"""A script for testing that TensorFlow is installed correctly on Windows.
The script will attempt to verify your TensorFlow installation, and print
suggestions for how to fix your installation.
"""

import ctypes
import imp
import sys

def main():
  try:
    import tensorflow as tf
    print("TensorFlow successfully installed.")
    if tf.test.is_built_with_cuda():
      print("The installed version of TensorFlow includes GPU support.")
    else:
      print("The installed version of TensorFlow does not include GPU support.")
    sys.exit(0)
  except ImportError:
    print("ERROR: Failed to import the TensorFlow module.")

  candidate_explanation = False

  python_version = sys.version_info.major, sys.version_info.minor
  print("\n- Python version is %d.%d." % python_version)
  if not (python_version == (3, 5) or python_version == (3, 6)):
    candidate_explanation = True
    print("- The official distribution of TensorFlow for Windows requires "
          "Python version 3.5 or 3.6.")
  
  try:
    _, pathname, _ = imp.find_module("tensorflow")
    print("\n- TensorFlow is installed at: %s" % pathname)
  except ImportError:
    candidate_explanation = False
    print("""
- No module named TensorFlow is installed in this Python environment. You may
  install it using the command `pip install tensorflow`.""")

  try:
    msvcp140 = ctypes.WinDLL("msvcp140.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'msvcp140.dll'. TensorFlow requires that this DLL be
  installed in a directory that is named in your %PATH% environment
  variable. You may install this DLL by downloading Microsoft Visual
  C++ 2015 Redistributable Update 3 from this URL:
  https://www.microsoft.com/en-us/download/details.aspx?id=53587""")

  try:
    cudart64_80 = ctypes.WinDLL("cudart64_80.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'cudart64_80.dll'. The GPU version of TensorFlow
  requires that this DLL be installed in a directory that is named in
  your %PATH% environment variable. Download and install CUDA 8.0 from
  this URL: https://developer.nvidia.com/cuda-toolkit""")

  try:
    nvcuda = ctypes.WinDLL("nvcuda.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'nvcuda.dll'. The GPU version of TensorFlow requires that
  this DLL be installed in a directory that is named in your %PATH%
  environment variable. Typically it is installed in 'C:\Windows\System32'.
  If it is not present, ensure that you have a CUDA-capable GPU with the
  correct driver installed.""")

  cudnn5_found = False
  try:
    cudnn5 = ctypes.WinDLL("cudnn64_5.dll")
    cudnn5_found = True
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'cudnn64_5.dll'. The GPU version of TensorFlow
  requires that this DLL be installed in a directory that is named in
  your %PATH% environment variable. Note that installing cuDNN is a
  separate step from installing CUDA, and it is often found in a
  different directory from the CUDA DLLs. You may install the
  necessary DLL by downloading cuDNN 5.1 from this URL:
  https://developer.nvidia.com/cudnn""")

  cudnn6_found = False
  try:
    cudnn = ctypes.WinDLL("cudnn64_6.dll")
    cudnn6_found = True
  except OSError:
    candidate_explanation = True

  if not cudnn5_found or not cudnn6_found:
    print()
    if not cudnn5_found and not cudnn6_found:
      print("- Could not find cuDNN.")
    elif not cudnn5_found:
      print("- Could not find cuDNN 5.1.")
    else:
      print("- Could not find cuDNN 6.")
      print("""
  The GPU version of TensorFlow requires that the correct cuDNN DLL be installed
  in a directory that is named in your %PATH% environment variable. Note that
  installing cuDNN is a separate step from installing CUDA, and it is often
  found in a different directory from the CUDA DLLs. The correct version of
  cuDNN depends on your version of TensorFlow:
  
  * TensorFlow 1.2.1 or earlier requires cuDNN 5.1. ('cudnn64_5.dll')
  * TensorFlow 1.3 or later requires cuDNN 6. ('cudnn64_6.dll')
    
  You may install the necessary DLL by downloading cuDNN from this URL:
  https://developer.nvidia.com/cudnn""")
    
  if not candidate_explanation:
    print("""
- All required DLLs appear to be present. Please open an issue on the
  TensorFlow GitHub page: https://github.com/tensorflow/tensorflow/issues""")

  sys.exit(-1)

if __name__ == "__main__":
  main()
~~~

## 2. 텐서플로우 설치 [^tensorflow-rstudio] {#install-tensorflow}

[^tensorflow-rstudio]: [TensorFlow for R from RStudio - Installing TensorFlow](https://tensorflow.rstudio.com/tensorflow/articles/installation.html)

CPU 버젼 텐서플로우 설치는 `tensorflow` 팩키지를 설치하고 나서, `install_tensorflow()` 명령어로 설치하면 된다.
파이썬 언어에 기초하여 개발되었기 때문에 다수 파이썬 설치방법을 지원하고 있다.

|  설치방법  |                        설명                                 |
|:----------:|:------------------------------------------------------------|
| auto       | 현재 운영체제 기본설정에 맞춰 자동 설치                     |
| virtualenv | `~/.virtualenvs/r-tensorflow` 위치한 파이썬 가상환경에 설치 |
| conda      | `r-tensorflow` 명칭으로 명명된 아나콘다 파이썬 환경에 설치  |
| system     | 시스템 파이썬 환경에 설치                                   |

단, “virtualenv”, “conda” 는 리눅스와 맥OSX를 지원하고, “conda”, “system”은 윈도우에 적용된다.
예를 들어, `install_tensorflow(method = "conda")` 방식으로 설치할 수 있다.


~~~{.r}
# install.packages("tensorflow")

library(tensorflow)
#install_tensorflow()
~~~

R에서 텐서플로우를 사용할 준비가 되었다면 `Hello, TensorFlow!`를 실행하여 정상적으로 동작하는지 확인해본다.


~~~{.r}
sess = tf$Session()
hello <- tf$constant('Hello, TensorFlow!')
sess$run(hello)
~~~



~~~{.output}
b'Hello, TensorFlow!'

~~~
