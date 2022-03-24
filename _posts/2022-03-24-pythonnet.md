---
title: pythonnet 사용법(.NET Core, .NET Framework)
author: Jandari
date: 2021-03-24 17:04:00 -0500
categories: [.NET]
tags: [.NET, pythonnet]
math: true
mermaid: true
---

* [https://github.com/pythonnet/pythonnet](https://github.com/pythonnet/pythonnet)
* [pythonnet Wiki](https://github.com/pythonnet/pythonnet/wiki)

Python.NET은 Python을 CLR과 상호 작용하도록 만들어진 Nuget 패키지 입니다.

즉, C# 코드에서 Python 코드를 사용 할 수 있습니다.

2022-03-24 기준 Python 3.8 까지 지원합니다.


## pythonnet 설치

현재 3.8까지 지원하기 때문에 Windows PC에 Python 3.8을 설치 합니다.

python 3.8에 설치 된 pythonnet 및 numpy 등 여러 패키지는 .NET Framework, .NET Core 6에서 함께 사용 가능 합니다.

[Python Download](https://www.python.org/downloads/)

Python을 다운 받은 후 pip로 pythonnet을 설치합니다.

```
> python -m pip install --upgrade pip

Requirement already satisfied: pip in c:\python\python38\lib\site-packages (21.1.1)
Collecting pip
  Using cached pip-22.0.4-py3-none-any.whl (2.1 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 21.1.1
    Uninstalling pip-21.1.1:
      Successfully uninstalled pip-21.1.1
  WARNING: The scripts pip.exe, pip3.8.exe and pip3.exe are installed in 'C:\Python\Python38\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-22.0.4
```

<br/>

공식 사이트의 예제에서 numpy를 사용하므로 numpy도 설치하여 줍니다.

```
> python -m pip install numpy

Collecting numpy
  Using cached numpy-1.22.3-cp38-cp38-win_amd64.whl (14.7 MB)
Installing collected packages: numpy
  WARNING: The script f2py.exe is installed in 'C:\Python\Python38\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed numpy-1.22.3
```

<br/>

```
> python -m pip install pythonnet

Collecting pythonnet
  Using cached pythonnet-2.5.2-cp38-cp38-win_amd64.whl (81 kB)
Collecting pycparser
  Using cached pycparser-2.21-py2.py3-none-any.whl (118 kB)
Installing collected packages: pycparser, pythonnet
Successfully installed pycparser-2.21 pythonnet-2.5.2
```

pythonnet을 설치하고 나면 `{Python 설치경로}\Lib\site-packages`에 `Python.Runtime.dll`이 있는 것을 확인 할 수 있다.

## .NET Framework 사용하기

#### C# 프로젝트에 pythonnet_py38_win 설치

Visual Studio에서 .NET Framework 프로젝트를 하나 생성 후 Nuget에서 pythonnet_py38_win을 설치해 준다.

```
PM> nuget install pythonnet_py38_win
```

설치 후 아래와 같이 Programs.cs에 작성합니다.

[pythonnet 공식 예제](https://github.com/pythonnet/pythonnet/blob/master/README.rst)


```cs
using Python.Runtime;
using System;
using System.Collections.Generic;


namespace ConsoleApp1
{
    internal class Program
    {
        static void Main(string[] args)
        {
            using (Py.GIL())
            {
                dynamic np = Py.Import("numpy");
                Console.WriteLine(np.cos(np.pi * 2));

                dynamic sin = np.sin;
                Console.WriteLine(sin(5));

                double c = (double)(np.cos(5) + sin(5));
                Console.WriteLine(c);

                dynamic a = np.array(new List<float> { 1, 2, 3 });
                Console.WriteLine(a.dtype);

                dynamic b = np.array(new List<float> { 6, 5, 4 }, dtype: np.int32);
                Console.WriteLine(b.dtype);

                Console.WriteLine(a * b);
                Console.ReadKey();
            }
        }
    }
}

```

```
1.0
-0.958924274663
-0.6752620892
float64
int32
[  6.  10.  12.]
```

## .NET Core 사용하기

#### C# 프로젝트에 pythonnet_netstandard_py38_win 설치

Visual Studio에서 .NET Framework 프로젝트를 하나 생성 후 Nuget에서 `pythonnet_netstandard_py38_win`을 설치해 준다.

```
PM> nuget install pythonnet_netstandard_py38_win
```

설치 후 아래와 같이 Programs.cs에 작성합니다.

[pythonnet 공식 예제](https://github.com/pythonnet/pythonnet/blob/master/README.rst)


```cs
using Python.Runtime;

using (Py.GIL())
{
    dynamic np = Py.Import("numpy");
    Console.WriteLine(np.cos(np.pi * 2));

    dynamic sin = np.sin;
    Console.WriteLine(sin(5));

    double c = (double)(np.cos(5) + sin(5));
    Console.WriteLine(c);

    dynamic a = np.array(new List<float> { 1, 2, 3 });
    Console.WriteLine(a.dtype);

    dynamic b = np.array(new List<float> { 6, 5, 4 }, dtype: np.int32);
    Console.WriteLine(b.dtype);

    Console.WriteLine(a * b);
    Console.ReadKey();
}



```

```
1.0
-0.958924274663
-0.6752620892
float64
int32
[  6.  10.  12.]
```