---
title: WPF Binding의 RelativeSource 속성 
author: Jandari
date: 2022-03-30 08:30:00 -0500
categories: [.NET, WPF]
tags: [.NET, C#, WPF, MVVM]
math: true
mermaid: true
---

## 소개

WPF에서 Binding의 태그 중 RelativeSource 속성은 바인딩 할 객체를 찾을 때 사용됩니다.

#### 자기 자신

```xml
<TextBlock Text="{Binding RelativeSource={RelativeSource self}, Path=FontFamily}" />
```

자기 자신이 바인딩 소스인 경우 self로 자정하고 Path로 어떤 것을 바인딩 할 지 지정하면 됩니다.

#### 상위 객체

```xml
<StackPanel Orientation="Horizontal">
    <TextBlock Text="{Binding Path=Orientation, RelativeSource={RelativeSource AncestorType={x:Type StackPanel}}}"/>
</StackPanel>
```

자신의 부모 객체의 경우 감싸고 있는 Control의 이름을 `AncestorType`로 지정해주면 됩니다.

하지만 StackPanel이 StackPanel로 감싸고 있을 경우는 어떻게 할까요??


```xml
<StackPanel Orientation="Vertical">
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="{Binding Path=Orientation, RelativeSource={RelativeSource AncestorType={x:Type StackPanel}, AncenstorLevel=2}}"/>
    </StackPanel>
</StackPanel>
```

위와 같을 때는 `AncenstorLevel=2`처럼 몇번 째 부모 Control인지 지정해주면 됩니다.

#### ControlTemplate 정의 엘리먼트 일 경우

```xml
<ControlTemplate TargetType="{x:Type Button}"> 
    <Border x:Name="_pBorder" BorderThickness="2" BorderBrush="Black" CornerRadius="20"> 
        <Border.Background> 
            <LinearGradientBrush StartPoint="0 0.5" EndPoint="1 0.5">
                <GradientStop Offset="0.0" Color="{Binding RelativeSource={RelativeSource TemplatedParent}, Path=Background.Color}" />
                <GradientStop Offset="0.9" Color="White" />
            </LinearGradientBrush>
        </Border.Background>
        <ContentPresenter Margin="2" HorizontalAlignment="Center" VerticalAlignment="Center" RecognizesAccessKey="True" />
    </Border>
</ControlTemplate>
```

#### x:Name으로 Control 찾기

```xml
<StackPanel x:Name="Panel1" Orientation="Vertical">
    <StackPanel x:Name="Panel2" Orientation="Horizontal">
            <TextBlock Text="{Binding Path=Orientation, ElementName=Panel1}"/>
        </StackPanel>
</StackPanel>
<TextBlock Text="{Binding Path=Orientation, ElementName=Panel2}"/>
```

위와 같이 상대 경로로 찾지 않고 ElementName으로 바로 찾을 수 있습니다.