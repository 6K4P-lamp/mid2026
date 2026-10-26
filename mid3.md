### 문제
\[mid3.md\] 숫자야구 판정 게임을 작성하시오.

### 프롬프트
C로 숫자야구 게임을 작성해줘. 코드는 간결하게

### 코드
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    int secret[3], guess[3];
    int strikes, balls, tries = 0;

    srand(time(NULL));

    // 중복 없는 3자리 랜덤 숫자 생성 (1~9)
    for (int i = 0; i < 3; i++) {
        secret[i] = rand() % 9 + 1;
        for (int j = 0; j < i; j++) {
            if (secret[i] == secret[j]) { i--; break; }
        }
    }

    printf("--- 숫자야구 게임 시작 (1~9, 중복 없음) ---\n");

    do {
        strikes = balls = 0;
        printf("\n3개의 숫자를 띄워서 입력하세요 (예: 1 2 3): ");
        if (scanf_s("%d %d %d", &guess[0], &guess[1], &guess[2]) != 3) break;
        tries++;

        // 스트라이크 및 볼 판정
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (guess[i] == secret[j]) {
                    if (i == j) strikes++;
                    else balls++;
                }
            }
        }

        printf("결과: %d 스트라이크, %d 볼\n", strikes, balls);

    } while (strikes < 3);

    printf("\n축하합니다! %d번 만에 정답을 맞췄습니다.\n", tries);
    return 0;
}
```

<img width="1109" height="592" alt="image" src="https://github.com/user-attachments/assets/3810c067-32d1-45c0-b16d-c77061012b4f" />
