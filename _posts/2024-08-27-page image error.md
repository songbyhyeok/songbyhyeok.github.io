---
title: Github page 이미지 엑박 현상
categories: github
---

## 이슈
글 작성 후 이미지 엑박으로 업로드가 되지 않는 현상이 발생하였다.

## 로컬과 서버 모두 이미지가 출력이 되지 않는다.
![image](https://github.com/user-attachments/assets/ca6188ee-99b1-4541-942f-641ab4e8bdba)
![image](https://github.com/user-attachments/assets/53e2cb04-994d-4d1d-95d8-db39474d9487)

사진과 같이 서버와 로컬 두 환경에서 모두 이미지가 나타나지 않았고, 평소 이미지 업로드 방식이 문제가 되었나 생각을 했다. 그러나 다른 글에서의 이미지 url은 현재 엑박 현상이 나타난 글에서 출력에 이상이 없었기에 방식에서의 차이는 아니라고 생각을 했다. 그리고 나와 같은 문제를 겪는 사람들의 이야기를 들어보니 대부분 절대 및 상대 경로의 문제만 거론할 뿐 다른 해결 방안에 대해서는 언급하지 않았다. 그러던 도중 gpt씨가 계속해서 파일의 공개 상태 및 접근성에 대해서 강조하고 있었고 난 걸리던 부분이 하나 있었는지 아차 싶었다.

## 이미지도 접근성이란 게 있나 보다.
내가 사용하던 이미지 업로드 방식은 깃허브 저장소 이슈 탭에서 url을 생성해 가져다 사용하는 것인데, 문제는 저장소가 private였다. 이점이 이미지 url의 접근성을 폐쇄할 수 있겠다 싶었고, 공개 저장소로 바꾸어 url을 재생성해 보았다.
<img src="https://github.com/user-attachments/assets/ed3a5421-2ecd-44a6-9ad3-49f8fe62821b" alt="Description" width="70%"/>

그림과 같이 잘 해결되었다~

