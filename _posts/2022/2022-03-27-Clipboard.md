---
title: C# Clipboard 및 Serialize
author: Jandari
date: 2022-03-28 17:34:00 -0500
categories: [.NET, WPF]
tags: [.NET, C#, WPF, MVVM]
math: true
mermaid: true
---

## 개요

이 포스트는 WPF-MVVM에서 UI의 색상을 복사하여 붙여넣기 예제 프로그램 및 설명입니다.

C#에서 클립보드를 사용하기 위해서는 `System.Windows`의 Clipboard Class를 사용합니다.

#### 복사하기

클립보드에 복사하기 위해서는 SetDataObject를 사용합니다.

`Clipboard.SetDataObject(object data, bool copy)`

두번째 매개 변수인 copy의 경우 true 이면 프로그램을 종료하여도 클립보드에 계속 남아 있고 false 인 경우 프로그램이 종료하면 클립보드의 데이터도 같이 없어집니다.

```cs
Clipboard.Clear();
var dataObject = new DataObject();
dataObject.SetData("SelectedColor", SelectedColor, true);
Clipboard.SetDataObject(dataObject, true);
```

클립보드에 데이터를 넣을 때는 DataObject를 통해 구분이 가능하도록 넣을 수 있습니다.

DataObject의 dataObject.SetData() 의 경우 그냥 넣어도 되고 format 을 지정하여 구분을 가능하도록 하여 여러 데이터를 넣고 원하는 것만 빼서 쓸 수 있습니다.

예제에서는 SelectedColor라는 format을 지정하여 사용하였습니다.

## 예제

[CopyTest.zip](/assets/img/post/2022-03-27-Clipboard/CopyTest.zip)