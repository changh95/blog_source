---
title: 효율적인 이미지 버퍼 구현 (C++17 멀티 쓰레드)
date: 2021-06-25 14:08:46
tags: 
- SLAM
- Visual-SLAM
- Thread
- Image processing
categories: 
- [SLAM, 공부 자료]
excerpt: Thread기반 이미지 버퍼 구현
---

## Naive 구현 - Sequential method

OpenCV를 이용해서 실시간 이미지 프로세싱을 구현할 때, 우리는 보통 `cv::VideoCapture` 기능을 이용한다. 이 기능은 두가지 기능을 지원하는데, 첫째는 기존에 촬영해둔 이미지 데이터셋을 읽어들이는 것이고, 둘째는 PC에 연결된 카메라로부터 실시간 비디오 스트림을 받아오는 것이다.

종종 실시간 프레임~프레임 트랙킹 알고리즘을 구현할 때, 프레임간의 간격이 짧은 (i.e. FPS가 높은) 데이터가 요구된다.

데이터셋을 기반으로 개발을 하고 있다면 어렵지 않게 높은 FPS를 얻을 수 있다. 단순히 높은 FPS의 카메라로 촬영을 해두고, 개발은 '이미지 읽기 -> 알고리즘 돌리기 -> 다음 이미지 읽기 -> 알고리즘 돌리기' 를 하면 되기 때문이다. 알고리즘이 실시간에 돌아가던, 10초씩 돌아가던 우리의 이미지는 여전히 실시간의 데이터를 담고 있다.

카메라로부터 실시간 데이터를 가져오는 경우는 좀 더 복잡하다. 이미지를 읽고 -> 알고리즘을 수행하고 -> 이미지를 읽는... 이런 형태의 파이프라인을 가지고 있다면, 알고리즘을 수행하는데에 걸리는 시간이 길어질수록 그 다음 이미지를 취득하는데에 더 오래 걸릴 것이다. 이 경우, 개발 단계에서도 알고리즘이 실시간성을 유지해야하기 되는데, 대부분 개발 초기단계에서는 최적화가 되지 않은 코드가 많기 때문에 개발이 어렵게 된다.

실시간 웹캠을 이용하면 데이터셋 기반 방식보다 더 다양한 실험을 빠르게 할 수 있다는 장점이 있기 때문에 이 방법을 포기하기에는 너무 아깝다. 실시간으로 데이터를 받으면서, 알고리즘 실험은 그대로 할 수 있으면 어떨까?

## Thread 기반 구현

아래 방법은 C++ 멀티쓰레딩 방식을 이용해서 실시간 이미지를 받아서 메모리에 저장하는 코드 구현이다. 2개의 쓰레드가 동시에 돌아가는데, 첫번째는 `capture()`를 실행하는 t1 쓰레드, 두번째는 `process()`를 실행하는 t2 쓰레드이다.

t1은 독립된 쓰레드에서 `cv::VideoCapture` 기반으로 카메라에서 이미지를 읽는다. 읽은 이미지는 `buffer`라는 queue에 순차적으로 저장된다.

t2는 다른 쓰레드에서 수행되는 작업이다. 실시간 트랙킹 알고리즘 등이 이 쓰레드에 구현될 수 있다. 아래 코드에서는 간단하게 rgb2gray가 구현되었고, 시간 제약을 두기 위해 waitKey에 40ms를 넣었다. 이미지를 읽고나

```cpp
std::queue<cv::Mat> buffer;
std::mutex mutex;
bool isRunning = false;

void rgb2gray()
{
    cv::Mat frameGray;

    while (true)
    {
        if (buffer.empty())
        {
            if (!isRunning) // Program termination
                break;

            continue;
        }

        mutex.lock();

        auto img = buffer.front();
        buffer.pop();
        std::cout << "Pop:" << buffer.size() << std::endl;

        mutex.unlock();

        cv::cvtColor(img, frameGray, cv::COLOR_RGB2GRAY);
        cv::imshow("lol", frameGray);
        if (27 == cv::waitKey(40))
            isRunning = false;
    }
}

void capture()
{
    cv::VideoCapture cap(0);
    cap.open(0);
    cv::Mat frame;
    cv::Mat frameClone;

    while (isRunning)
    {

        // if (buffer.size() > 100)
        //     buffer.pop();

        cap.read(frame);
        frameClone = frame.clone();

        std::scoped_lock lock(mutex);
        buffer.push(frameClone);
        std::cout << buffer.size() << std::endl;
    }
}

int main()
{
    isRunning = true;

    std::thread t1(&capture);
    std::thread t2(&process);

    t1.join();
    t2.join();

    return 0;
}
```
