---
title: First Class Collection(일급 컬렉션)
author: Jandari
date: 2022-04-25 19:00:00 -0500
categories: [DDD]
tags: [DDD, 일급 컬렉션]
math: true
mermaid: true
---

## 참조
* [객체지향 생활 체조 9가지](https://otrodevym.tistory.com/entry/CS-OOP-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%83%9D%ED%99%9C-%EC%B2%B4%EC%A1%B0-9%EA%B0%80%EC%A7%80)
* [First Class Collection](https://wickso.me/java/first-class-collection/)

<br>

## 소개
최근 DDD를 공부하면 `일급 컬렉션`이라는 단어가 나와 찾아보게 되었습니다.

`일급 컬렉션`은 마틴 파울러가 쓴 [소트웍스 앤솔러지 : 소프트웨어 기술과 혁신에 관한 에세이](http://www.yes24.com/Product/Goods/3290339)의 객체지향 생활체조 파트에서 언급되는 8번째 규칙입니다.

![image](/assets/img/post/2022-04-25-FirstClassCollection/1.jpg)

<br>

## 일급 컬렉션이란?

Collection을 Wrapping하면서, 그 외 다른 변수가 없는 클래스의 상태를 일급 컬렉션이라고 합니다.

예를 들어 아래의 좌측 코드를 우측 코드로 변경 된 것을 일급 컬렉션이라고 합니다.

![image](/assets/img/post/2022-04-25-FirstClassCollection/2.jpg)

그냥 봐서는 Emails라는 Class는 왜 만들어서 코드의 양만 늘어나게 했는지 의아해 할 수도 있습니다.

일급 켈렉션의 이점은 아래 같습니다. 

<br>

### 1. 비즈니스에 종속적인 자료구조

* 상위 클래스(User)에서 컬렉션을 선언하게 되면 해당 컬렉션이 필요한 모든 장소에서 검증 로직이 들어가게 됩니다.
* 비즈니스에 종속적이라는 말은 생성 된 클래스의 컬렉션을 관리하는 클래스(일급 컬렉션)을 따로 만들어서 [+클래스 내부적으로 비즈니스 검증 로직을 관리 할 수 있다는 말입니다.+]

#### As-Is

* User는 Email에 대해 아래와 같은 비즈니스를 가집니다.
  * Email은 최대 10개 까지 가질 수 있다.
  * Email은 중복값이 없어야 한다.
* 위와 같은 비즈니스가 있을 때 아래와 같이 작성 할 수 있습니다.

```cs
public class User
{
    private List<Email> _emails = new List<Email>();

    public bool AddEmail(Email email)
    {
        if (_emails.Count >= 10)
            return false;

        if (_emails.Any(x => x.Equals(email)))
            return false;

        _emails.Add(email);
        return true;
    }
}
```


#### To-Be

* 일급 컬렉션을 사용하면 이메일 관련 요구사항은 Emails 에서만 관리하게 되면서 응집도(Cohesion)를 높히고 User 와 Email 에 대한 커플링(Coupling)을 낮출 수 있습니다.

```cs
public class User
{
    private Emails _emails { get; set; }

    public bool AddEmail(Email email)
    {
        return _emails.AddEmail(email);
    }
}

public class Emails
{
    private List<Email> _emails = new List<Email>();

    public bool AddEmail(Email email)
    {
        if (_emails.Count >= 10)
            return false;

        if (_emails.Any(x => x.Equals(email)))
            return false;

        _emails.Add(email);
        return true;
    }
}
```

<br>

### 2. Collection의 불변성을 보장

* 일급 컬렉션은 컬렉션의 불변을 보장하는데, 단순히 readonly을 사용하는 것이 아니라 캡슐화를 통해 이뤄집니다.
* readonly은 재할당만 금지할 뿐 Add, Remove가 가능합니다.
* 하지만 일급 컬렉션에서 getter만 선언하고 setter를 생성하지 않으면 불변 컬렉션이 됩니다.

```cs
public class Emails
{
    private readonly List<Email> _emails;

    public Emails(List<Email> emails)
    {
        _emails = emails;
    }

    public Email this[int index] { get => new Email(_emails[index].Local, _emails[index].Domain); }
}
```

<br>

### 3. 상태와 행위를 한 곳에서 관리
* 일급 컬렉션을 사용하면 값과 로직이 함께 존재 하기 때문에 응집도가 높아집니다.
* Emails 컬렉션을 사용하게 되면 똑같은 기능이 중복 되지 않고 한 곳에서 로직 변경이 가능합니다.

#### As-Is
* 만약 Email 컬렉션을 사용하는 객체 Manager가 늘었다고 가정 합니다.
* User객체와 Manager 객체의 Email 관리 비즈니스가 중복 되게 됩니다.

```cs
public class User
{
    private List<Email> _emails { get; set; }
    public bool AddEmail(Email email)
    {
        if (_emails.Count >= 10)
            return false;

        if (_emails.Any(x => x.Equals(email)))
            return false;

        _emails.Add(email);
        return true;
    }
}
public class Manager
{
    private List<Email> _emails { get; set; }
    public bool AddEmail(Email email)
    {
        if (_emails.Count >= 10)
            return false;

        if (_emails.Any(x => x.Equals(email)))
            return false;

        _emails.Add(email);
        return true;
    }
}
```

#### To-Be
* 일급 컬렉션을 사용하게 되면 응집도가 높아지기 때문에 비즈니스 로직이 변경 되어도 Emails Class만 수정하면 됩니다.

```cs

public class User
{
    private Emails _emails { get; set; }

    public bool AddEmail(Email email)
    {
        return _emails.AddEmail(email);
    }
}
public class Manager
{
    private Emails _emails { get; set; }
    public bool AddEmail(Email email)
    {
        return _emails.AddEmail(email);
    }
}

public class Emails
{
    private readonly List<Email> _emails;

    public Emails()
    {
        _emails = new List<Email>();
    }

    public bool AddEmail(Email email)
    {
        if (_emails.Count >= 10)
            return false;

        if (_emails.Any(x => x.Equals(email)))
            return false;

        _emails.Add(email);
        return true;
    }
}

```

<br>

### 4. 이름이 있는 컬렉션

* 일급 컬렉션을 사용하게 되면 변수에만 도출되던 비즈니스가 컬렉션 자체에서 도출이 가능합니다.
* 예를 들어 Naver Emails에 대한 요구사항을 검색하거나 선언 할 경우 아래와 같은 문제점을 겪을 수 있습니다.
  * 개발자에 따라 변수명이 다르다.
  * 중요한 값이지만 명확하게 표현해둔 단어/변수명이 없다.
* 일급 컬렉션을 사용하면 Naver Email에 대한 요구사항이 바뀌었을 때 `NaverEmails`을 참조하는 코드를 모두 찾을 수 있다.

#### As-Is

```cs
public class User
{
    private List<Email> _mireroEmails { get; set; }
    private List<Email> _naverEmails { get; set; }
}
```


#### To-Be

```cs
public class User
{
    private MireroEmails _mireroEmails { get; set; }
    private NaverEmails _naverEmails { get; set; }
}
```

<br>

## 예제 코드

* [FirstClassCollection.zip](/assets/img/post/2022-04-25-FirstClassCollection/FirstClassCollection.zip)