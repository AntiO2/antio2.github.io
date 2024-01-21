---
title: "A+B Problem"
description: 
date: 2017-03-08
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
categories:
---

```CPP
#include<bits/stdc++.h>
using namespace std;
int a,b,ans;
int main()
{
cin>>a>>b;
if(a==b)
{
ans=a*4-b*2;
}
if(a>b)
{
ans=a+a+a+b+b+b;
ans=ans/3;
}
if(a<b)
{
ans=a+b+b+a;
ans=ans/2;
}
cout<<ans;
}
```

