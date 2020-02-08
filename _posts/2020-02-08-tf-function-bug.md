---
layout: post
title:  "tf.function bug"
date:   2020-02-08 09:00:03
categories: tools
---



# tf.function bug



그동안 장대한 포스트만 쓰다가 이번엔 짧은 버그 리포트 하나를 써보려 한다.  

Tensorflow에서 `@tf.function`이라는 유용한 데코레이터가 있다.  

그런데 이걸 쓰다가 계속 *WARNING*이 떠서 이거 왜 이러나... 찾아보고 테스트 하다가 원인을 발견한 것을 기록하여 같은 실수를 막고자 한다.



## AutoGraph could not transform

어떤 함수에 `@tf.function`를 붙여서 실행을 해보니 "**AutoGraph could not transform**'로 시작하는 경고 메시지가 시작할 때 뿐만 아니라 학습 중에도 계속 발생했다.

`@tf.function` 안의 다른 함수들은 문제가 없는데 유독 특정 함수만 계속 경고가 떴다. 코드를 바꿔가면서 실험을 해보다가 아래와 같은 간단한 테스트 코드를 만들었다.



```python
import tensorflow as tf

class TestTfBug:
    def test_func_in_class(self):
        a = tf.random.uniform((8, 100, 100, 3))
        b = tf.random.uniform((8, 100, 100, 3))
        c = 0.5 * tf.reduce_mean(tf.abs(a), axis=[1, 2, 3]) + \
            0.5 * tf.reduce_mean(tf.abs(b), axis=[1, 2, 3])
        return c

def test_func_just_func():
    a = tf.random.uniform((8, 100, 100, 3))
    b = tf.random.uniform((8, 100, 100, 3))
    c = 0.5 * tf.reduce_mean(tf.abs(a), axis=[1, 2, 3]) + \
        0.5 * tf.reduce_mean(tf.abs(b), axis=[1, 2, 3])
    return c

# This function results in WARNING:tensorflow:AutoGraph could not transform ~~
@tf.function
def test_tf_bug_class():
    result = TestTfBug().test_func_in_class()

# This function has no problem
@tf.function
def test_tf_bug_func():
    result = test_func_just_func()

if __name__ == "__main__":
    test_tf_bug_class()
    test_tf_bug_func()
```

위에서 실행한 두 함수 중에 하나씩만 번갈아가며 실행하면 어느쪽에서 경고가 나오는지 알 수 있다. `test_tf_bug_class()`를 실행하면 다음과 같은 경고가 뜬다.

> WARNING:tensorflow:AutoGraph could not transform <bound method TestTfBug.test_func_in_class of <\__main__.TestTfBug object at 0x7f9da0181a50>> and will run it as-is.
> Please report this to the TensorFlow team. When filing the bug, set the verbosity to 10 (on Linux, `export AUTOGRAPH_VERBOSITY=10`) and attach the full output.
> Cause: expected exactly one node node, found [<gast.gast.FunctionDef object at 0x7f9da0176190>, <gast.gast.Return object at 0x7f9da01767d0>]



## 해결 방법

**아니 똑같은 함수를 실행했는데 그냥 함수에서는 문제가 없고 클래스 메소드에서만 문제가 생긴하는게 말이 되나?**  

그래서 이걸 텐서플로 이슈에 올려볼까 했더니 역시 이미 이슈에 있는 문제였다.  

<https://github.com/tensorflow/tensorflow/issues/35765>  

<https://github.com/tensorflow/tensorflow/issues/35810>  

이것은 `\`를 이용해 코드 라인을 나눌때 생기는 문제였다. 만약 코드를 다음과 같이 고치면 문제가 없다.

```python
import tensorflow as tf

class TestTfBug:
    def test_func_in_class(self):
        a = tf.random.uniform((8, 100, 100, 3))
        b = tf.random.uniform((8, 100, 100, 3))
        # No backslash here!!
        c = 0.5 * tf.reduce_mean(tf.abs(a), axis=[1, 2, 3]) + 0.5 * tf.reduce_mean(tf.abs(b), axis=[1, 2, 3])
        return c

# No problem here
@tf.function
def test_tf_bug_class():
    result = TestTfBug().test_func_in_class()

if __name__ == "__main__":
    test_tf_bug_class()
```

문제의 원인을 좀 따져본다면 `@tf.function`이라는게 eager execution으로 돌아가는 코드를 그래프 모드에서 실행할 수 있는 코드로 변환해 준다. 다음 튜토리얼에서 코드 생성에 대한 내용을 볼 수 있다.  

<https://www.tensorflow.org/guide/function#use_python_control_flow>

그래프 모드에서 `if ~ else ~` 같은게 있으면 static graph를 만들 수 없기 때문에 똑같은 기능을 그래프 모드에서 실행할 수 있는 코드를 자동으로 생성하는 것이다.  그런데 그 코드 생성 과정에서 저 `backslash(\)`가 문제를 일으키는 듯 하다. 코드 생성에서 backslash는 생각 못 했나보다.

암튼 **backslash만 안 쓰면 문제 해결!!**  

뭐... 이슈에 있기 때문에 아마 몇 달 지나서 새 버전 나오면 없어질 문제인것 같다.

\- 끝 -

