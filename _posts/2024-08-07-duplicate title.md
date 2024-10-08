---
title: Minimal Mistakes 테마 사용 시 home 제목이 중복되는 현상 해결하기
categories: github
---

## 동일한 블로그 제목이 두 개가 표시되는 현상
![화면 캡처 2024-08-07 160300](https://github.com/user-attachments/assets/ab8422bd-bee0-4635-b4a0-c3e4622dd5c8)

메인 페이지에서는 블로그 제목이 하나만 표시가 되어야 하는데, 중복된 제목 두 개가 같이 출력되는 이슈가 발생하였다.

## 해결 과정
![image](https://github.com/user-attachments/assets/7b352134-1c00-46ad-b09e-81f17bf845f3)

해당 프로젝트는 github.io의 jekyll theme인 minimal-mistakes이다. 보다시피 폴더 구조가 복잡하게 되어 있고, Jekyll에서 사용하는 Liquid 템플릿 언어에 대해 문외한이기 때문에 상당히 난감한 상태였다. 

### 동작 추적하기
메인 페이지가 어디서 출력이 되고 그 페이지의 블로그 제목을 어디서 출력을 시키고 있는지 경로 추적을 할 수만 있다면, 아마 해결할 수 있을 것이라고 직감적으로 떠올랐다.

![image](https://github.com/user-attachments/assets/9eb2611e-0c10-46eb-b5d8-4b66d8627ba5)
![image](https://github.com/user-attachments/assets/7c6c6718-ddd1-49a8-831e-a61e076fc222)
![image](https://github.com/user-attachments/assets/ba1a54a6-fd4c-4b50-be84-b98abe658d07)
![image](https://github.com/user-attachments/assets/ac0a1677-c175-456a-84cd-7622baa08123)

default에서 드디어 <head> 구조가 포함된 것을 확인하였다. 

![image](https://github.com/user-attachments/assets/eab156f3-a481-4218-8f8c-5868ff893eab)

태그 내 두 html 중 어느 것이 title 명을 출력시키는 것인지 확인하기 위해 시도해보니 head.html이 역할을 하고 있는 것을 확인하였다.

![image](https://github.com/user-attachments/assets/5be16155-3b3e-4b1e-ac5d-af2d016a88e6)

들어가보니 head.html의 seo.html이 title 부분을 담당하고 있었다.

## 해결
![image](https://github.com/user-attachments/assets/9dab8865-d663-467e-9d6e-4d9c33c71f3c)

    <!-- 문제의 중복 현상 코드 -->
    assign page_title = page.title | default: site.title | replace: '|', '&#124;'
    assign seo_title = page_title | append: " " | append: title_separator | append: " " | append: site.title | replace: '|', '&#124;'

    <!-- 수정 코드 -->
    assign page_title = page.title | default: site.title | replace: '|', '&#124;'
    if page.url == '/' or page.title == site.title
    assign seo_title = site.title | replace: '|', '&#124;'
    else
    assign seo_title = page_title | append: " " | append: title_separator | append: " " | append: site.title | replace: '|', '&#124;'
    endif

## 알게된 정보
1. %- 이게 무슨 언어의 문법일까?
Jekyll에서 사용하는 Liquid 템플릿 언어의 구문, 태그는 로직을 수행하는 데 사용(예: 조건문, 반복문, 변수 할당)
2. seo.html의 역할
seo.html 파일은 Jekyll 사이트의 SEO(검색 엔진 최적화)를 위한 메타 태그를 생성하는 역할
3. 데이터 값의 출처
* page.title: 현재 페이지의 제목
* site.title: 사이트 전체의 기본 제목(_config.yml 파일에서 설정)