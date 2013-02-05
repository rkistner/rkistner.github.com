---
layout: post
title: "Android builds on Travis CI"
description: ""
category: android
excerpt: my fist blog
tags: [android ci]
---
{% include JB/setup %}

## Continuous integration

While doing some projects in Ruby over the last few years, the concept of [test-driven development][1] grew on me.
Automatically running these tests in some continuous integration environment is a very important part of TDD when
other developers are involved, especially open-source projects.

[Travis CI][2] recently gained a lot of popularity. While originally designed for Ruby projects, it can now be used
for projects in almost any language. It has the advantage of very simple setup, requiring no maintenance, and being
able to test your project multiple environments in parallel. Furthermore, it is completely free for open-source
projects.

However, getting an Android project running on Travis requires more setup than a Ruby project, and it is therefore
no surprise that very few open-source Android projects in the wild are using Travis. Levi Wilson
[got an Android build working][8] last year, but the Travis environment changed a little since then, and no
integration tests were performed.

In this blog post I explain how I got the tests for an Android project running on Travis, complete with integration
tests running on multiple emulator images (different Android versions).

The complete example project for this post is on [github][3], and the build results are available on [Travis][4]. I
will keep on updating the code as I find better ways to do the CI.

## Building with Maven

The first step is to get the project running locally with Maven. Unfortunately, this is no simple task. I might write
a blog post about this in the future, but until then you can:

 1. Look at the [project setup][3] used for this post.
 2. Read the [official documentation][5].
 3. Look at the [official moreseflash sample][6].

The end result is that the entire project can be built and tested using a command such as
`mvn install -Pintegration-tests`, assuming the environment is setup and an emulator is running.

## The Travis configuration

Jumping right in, here is the complete .travis.yml file. The details are explained in the following sections.

{% highlight yaml %}
language: java
jdk: oraclejdk7
env:
    matrix:
    # android-16 is always included
    - ANDROID_SDKS=android-8            ANDROID_TARGET=android-8   ANDROID_ABI=armeabi
    - ANDROID_SDKS=android-10           ANDROID_TARGET=android-10  ANDROID_ABI=armeabi
    - ANDROID_SDKS=sysimg-16            ANDROID_TARGET=android-16  ANDROID_ABI=armeabi-v7a
    - ANDROID_SDKS=android-17,sysimg-17 ANDROID_TARGET=android-17  ANDROID_ABI=armeabi-v7a
before_install:
    # Install base Android SDK
    - sudo apt-get update -qq
    - if [ `uname -m` = x86_64 ]; then sudo apt-get install -qq --force-yes libgd2-xpm ia32-libs ia32-libs-multiarch; fi
    - wget http://dl.google.com/android/android-sdk_r21.0.1-linux.tgz
    - tar xzf android-sdk_r21.0.1-linux.tgz
    - export ANDROID_HOME=$PWD/android-sdk-linux
    - export PATH=${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools

    # Install required components.
    # For a full list, run `android list sdk -a --extended`
    # Note that sysimg-16 downloads the ARM, x86 and MIPS images (we should optimize this).
    - android update sdk --filter platform-tools,android-16,extra-android-support,$ANDROID_SDKS --no-ui --force

    # Create and start emulator
    - echo no | android create avd --force -n test -t $ANDROID_TARGET --abi $ANDROID_ABI
    - emulator -avd test -no-skin -no-audio -no-window &

before_script:
    - ./wait_for_emulator

script: mvn install -Pintegration-tests -Dandroid.device=test
{% endhighlight %}

A lot of the configuration is based off [RapidFTR][7].

### Installing the Android SDK

Travis provides us with a 64-bit Ubuntu installation running Java, but we have to install the Android SDK ourselves.
One gotcha is that we need ia32-libs to be able run the Android tools, but simply running `apt-get install ia32-libs`
gives dependency issues.

We then need to install the specific Android components we need. We also install emulators for the specific Android
environment we test on. Using the `--filter` option to `android update sdk`, we can pick which components we want to
install, instead of installing everything (this wastes precious minutes on Travis).
Unfortunately, the sysimg filters install the ARM, x86 and MIPS emulators, and there is no simple way to filter this
further (suggestions are welcome). We cannot use the specific id's for filters (as suggested in some posts), as these
may change when updates to the SDK are published.

In my experience the ARM emulators start up significantly faster than the x86 emulators on Travis (not sure why),
 so I use them exclusively.

### Running the emulator

We need to run the emulator in windowless mode - the `-no-window` option.


 [1]: http://en.wikipedia.org/wiki/Test-driven_development
 [2]: https://travis-ci.org
 [3]: https://github.com/embarkmobile/android-maven-example
 [4]: https://travis-ci.org/embarkmobile/android-maven-example
 [5]: http://code.google.com/p/maven-android-plugin/wiki/GettingStarted
 [6]: https://github.com/jayway/maven-android-plugin-samples/tree/master/morseflash
 [7]: https://github.com/rapidftr/RapidFTR---Android/blob/master/.travis.yml
 [8]: http://levi-wilson.blogspot.com/2012/06/maven-android-travis-ci-and-more.html
