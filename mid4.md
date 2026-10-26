### 문제
[mid4.md] 높이 N을 입력받아 중앙 정렬된 피라미드 형태의 문자 패턴을 출력하시오. 홀수 번째 행은 *, 짝수 번째 행은 #으로 출력합니다.
입출력 예시 (N=4): 피라미드 모양
```
     *
   ###
  *****
#######
```
### 답
```c
#include <stdio.h>

int main() {
	int a;
	scanf_s("%d", &a);
	for (int i = 0; i < a; i++) {
		for (int j = 0; j < a - i - 1; j++) printf(" ");
		for (int j = 0; j < (i + 1) * 2 - 1; j++) {
			if (i % 2) {
				printf("#");
			}
			else {
				printf("*");
			}
		}
		printf("\n");
	}
	return 0;
}
```

<img width="1114" height="451" alt="image" src="https://github.com/user-attachments/assets/02f331c8-83b0-4e56-8504-9cd25ed62bdd" />
