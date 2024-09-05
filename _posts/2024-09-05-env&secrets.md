---
title: 환경 변수와 secrets
categories: github_actions
---

workflow 작성하면서 secrets와 환경 변수 두 개를 때에 따라 사용해야 할 일이 많아서 어떻게 사용할 수 있는지 정리 목적으로 작성한다.

## secrets
![image](https://github.com/user-attachments/assets/ba21aa9a-0fac-4a17-8e0d-6c5abc3257b6)

**1. 별도의 환경 Secrets Group 설정**  
생성된 my env는 다른 envir secrets 혹은 일반 secrets와 공용해서 사용할 수 없다.

**2. 일반적인 Secrets**  
별도의 env와 같이 공용으로 사용할 수 없다.  

    # secrets 변수 참조
    command ${{ secrets.value }}
    echo `${{ secrets.value }}  

## env
    # env에 secrets 대입
    name:  
    env:    
        my_env: ${{ secrets.value }}
    with:
        my_env2: ${{ secrets.value2 }}

    # env 변수 참조
    ${my_env2}
    echo "${my_env}"