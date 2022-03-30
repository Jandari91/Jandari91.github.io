---
title: DataGrid ColumnItem의 버튼으로 해당 행 삭제
author: Jandari
date: 2021-03-30 21:30:00 -0500
categories: [.NET, WPF]
tags: [.NET, C#, WPF, MVVM]
math: true
mermaid: true
---

## 개요

이 포스트는 WPF-MVVM에서 DataGrid의 행들이 각각 버튼을 가지고 있고 그 버튼을 클릭 했을 때 해당 행이 삭제되는 예제입니다.

## WPF 프로젝트 생성

MVVM 패턴으로 하기 때문에 아래와 같이 파일을 생성합니다.

![](/assets/img/post/2022-03-30-wpf-RemoveItem-In-DataGrid/1.jpg)

|파일명|설명|
|-|-|
|DelegateCommand|ICommand의 구현체이며 Button의 Click 바인딩으로 사용합니다.|
|Item|DataGrid에 들어가는 데이터 입니다.|
|PropertyChangeHelper|INotifyPropertyChanged 구현체로 바인딩 객체의 변화에 대해 이벤트를 발생하여 UI를 갱신합니다.|

#### MainWindow.xaml

```xml
<Window x:Class="ItemRemove.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ItemRemove"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="30"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>
        <Grid Grid.Row="0" >
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="50"/>
            </Grid.ColumnDefinitions>
            <TextBox Grid.Column="0" Text="{Binding InputText}" Width="Auto"/>
            <Button Grid.Column="1" Command="{Binding InputCommand}" Content="Input" Width="50" />
        </Grid>

        <DataGrid Grid.Row="1" AutoGenerateColumns="False" CanUserAddRows="False"
                Margin="5" HorizontalAlignment="Stretch" VerticalAlignment="Stretch"
                RowBackground="#fff" FontWeight="Bold" Foreground="#525252"
                ScrollViewer.CanContentScroll="True" Height="390" MaxHeight="390"
                AlternatingRowBackground="#f2f2f2" BorderBrush="#000" BorderThickness="1"
                ScrollViewer.HorizontalScrollBarVisibility="Visible"
                ScrollViewer.VerticalScrollBarVisibility="Auto"
                  SelectedItem="{Binding Path=SelectedItem}"
                  ItemsSource="{Binding Path=Items}">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Id" Binding="{Binding Path='Id'}" IsReadOnly="True"/>
                <DataGridTextColumn Header="Value" Binding="{Binding Path='Value'}" IsReadOnly="True"/>

                <DataGridTemplateColumn Header="View">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="Remove" Command="{Binding Path=DataContext.RemoveCommand, 
                                RelativeSource={RelativeSource FindAncestor, AncestorType={x:Type DataGrid}}}"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>

    </Grid>
</Window>

```

DataTemplate은 데이터의 틀을 지정해주는 Template입니다. DataGrid에서 `View`라는 Column을 따로 Template을 만드는데 이때 DataTemplate으로 틀을 만들어주면 됩니다.

[RelativeSource란](/posts/wpf-RelativeSource/)

#### MainViewModel

```cs
public class MainViewModel : PropertyChangedHelper
{
    private int _itemIndex = 0;

    private string _inputText;
    public string InputText
    {
        get { return _inputText; }
        set { SetField(ref _inputText, value, "ItemsSource"); }
    }

    private ObservableCollection<Item> _items;
    public ObservableCollection<Item> Items
    {
        get { return _items; }
        set { SetField(ref _items, value, "ItemsSource"); }
    }

    private Item _selectedItem;
    public Item SelectedItem
    {
        get { return _selectedItem; }
        set { SetField(ref _selectedItem, value, "ItemsSource"); }
    }

    public ICommand RemoveCommand { get; set; }
    public ICommand InputCommand { get; set; }
    

    public MainViewModel()
    {
        Items = new ObservableCollection<Item>();
        RemoveCommand = new DelegateCommand(RemoveCommandAction);
        InputCommand = new DelegateCommand(InputCommandAction);
    }

    private void InputCommandAction()
    {
        if(InputText != null)
            Items.Add(new Item(_itemIndex++, InputText));
    }

    private void RemoveCommandAction()
    {
        if(SelectedItem != null)
            Items.Remove(SelectedItem);
    }
}
```

`InputCommandAction`은 InputText에 글자를 넣고 Input 버튼을 누르면 DataGrid에 행이 추가 됩니다.

그리고 추가 된 행에서 Remove 버튼을 누르면 선택 된 행을 DataGrid에서 삭제합니다.

## 예제

[ItemRemoveInDataGrid.zip](/assets/img/post/2022-03-30-wpf-RemoveItem-In-DataGrid/ItemRemoveInDataGrid.zip)