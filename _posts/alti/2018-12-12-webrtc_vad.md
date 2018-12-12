---
layout: post
title: cloudhopper ssl option 
---

## web rtc lib를 이용한 vad 추가

### vad 를 통하여 발화를 확인하고 audio 저장을 해야 하는 요구사항으로 인해 vad 를 살펴보기 시작하였는데,기존에 vad 를 테스트 한 코드를 보고 문의해 본 결과 webRtc 가 가장 품질이 좋았다는 의견을 들었다.이에 따라 로컬에서 webRtc를 테스트 해보고 소음이 발생할 경우만 오디오를 저장하도록 코드를 수정하였다.

<pre><code>
 interface WebRTCVadNative : Library {

    companion object {

        val instance = Native.loadLibrary("webrtc_vad", WebRTCVadNative::class.java)

        val init = instance.WebRtcSpl_Init()
    }

    fun WebRtcSpl_Init()

    fun WebRtcVad_Create(): Pointer

    fun WebRtcVad_Free(handle: Pointer)

    fun WebRtcVad_Init(handle: Pointer): Int

    fun WebRtcVad_set_mode(handle: Pointer, mode: Int): Int

    fun WebRtcVad_Process(handle: Pointer, fs: Int, audioFrame: Pointer, frameLength: Int): Int
}
</code></pre>

### 하지만 인텔리제이 에서는 정상적으로 동작하던 내용이 스프링부트를 jar 로 만들고, 이를 도커로 만들어서 동작시켜 보니 so native file 을 로딩을 못하여 계속 에러가 발생하였다.

   
### 검색을 좀 해보니 jar 파일 내부에 so 파일을 갖고 있어도 이를 압축을 풀고 사용할 수 있도록 하는것은 지원할 생각이 없다고 되어 있다. - 문제는 알고 있지만, 다른 바쁜일도 많고 하니 이전과 같은 방식으로 사용하라고 되어 있었다.-Djava.library.path= 를 설정하여 열심히 테스트 해봤지만 진행이 안되서, 고민 하고 검색해보니 -Djna.library.path= 도 있어서 사용해보니 jna 로 사용하는 패스는 사용이 되어 docker 로 파일 복사를 진행하여, 참조할 수 있도록 진행하니, 잘된다.

### Dockerfile so 파일 복사
<pre><code>
 COPY ./Docker/libwebrtc_vad.so /
</code></pre>

### 스프링부트 실행시 jna path 설정
<pre><code>
 -Djna.library.path="/"'
</code></pre>

### so 파일 로드를 못 할 줄은 생각 못하였지만, 하루 정도 시간 소비하고 native 호출에 대해서 잘 배웠다.
