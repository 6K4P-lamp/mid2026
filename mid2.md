### 문제
\[mid2.md\] SQLite를 활용한 간단한 DB를 설계하고 구현 하시오. (시나리오는 각자 만들어서 하시오, 반드시 3명 이상의 데이터를 넣고, 검색을 포함해야 합니다.)

### 코드
```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"

// 전체 조회 시 호출될 콜백 함수
static int callback(void* NotUsed, int argc, char** argv, char** azColName) {
    for (int i = 0; i < argc; i++) {
        printf("%s = %s\t", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    printf("\n");
    return 0;
}

int main() {
    sqlite3* db;
    char* err_msg = 0;
    int rc;

    // 1. 데이터베이스 연결 (student.db 파일이 없으면 자동 생성)
    rc = sqlite3_open("student.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터베이스 열기 실패: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    // 2. 테이블 생성 (students: id, name, score)
    char* sql_create = "CREATE TABLE IF NOT EXISTS students(" \
        "id INTEGER PRIMARY KEY AUTOINCREMENT, " \
        "name TEXT NOT NULL, " \
        "score INTEGER);";

    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "테이블 생성 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // 3. 3명 이상의 데이터 추가 (INSERT)
    char* sql_insert = "INSERT INTO students (name, score) VALUES ('김철수', 85);" \
        "INSERT INTO students (name, score) VALUES ('이영희', 92);" \
        "INSERT INTO students (name, score) VALUES ('박민수', 78);";

    // 데이터가 이미 중복으로 쌓이는 것을 막으려면 생략하거나 별도로 처리할 수 있지만,
    // 과제 시연용으로는 위와 같이 간단히 실행해도 무방합니다.
    sqlite3_exec(db, sql_insert, 0, 0, &err_msg);

    // 전체 학생 목록 출력
    printf("=== 전체 학생 목록 ===\n");
    char* sql_select = "SELECT * FROM students;";
    sqlite3_exec(db, sql_select, callback, 0, &err_msg);
    printf("------------------------\n");

    // 4. 검색 기능 구현 (사용자로부터 이름을 입력받아 LIKE 검색)
    char search_name[50];
    printf("검색할 학생 이름을 입력하세요: ");
    scanf("%49s", search_name);

    // Prepared Statement를 사용하여 안전하게 쿼리 작성 (SQL Injection 방지)
    char* sql_search = "SELECT * FROM students WHERE name LIKE ?;";
    sqlite3_stmt* stmt;

    rc = sqlite3_prepare_v2(db, sql_search, -1, &stmt, 0);
    if (rc == SQLITE_OK) {
        // 검색어에 와일드카드(%) 추가
        char pattern[60];
        snprintf(pattern, sizeof(pattern), "%%%s%%", search_name);
        sqlite3_bind_text(stmt, 1, pattern, -1, SQLITE_STATIC);

        printf("\n=== 검색 결과 ===\n");
        int found = 0;
        while (sqlite3_step(stmt) == SQLITE_ROW) {
            found = 1;
            int id = sqlite3_column_int(stmt, 0);
            const unsigned char* name = sqlite3_column_text(stmt, 1);
            int score = sqlite3_column_int(stmt, 2);
            printf("ID: %d | 이름: %s | 점수: %d점\n", id, name, score);
        }
        if (!found) {
            printf("일치하는 학생을 찾을 수 없습니다.\n");
        }
        sqlite3_finalize(stmt);
    }
    else {
        fprintf(stderr, "쿼리 준비 실패: %s\n", sqlite3_errmsg(db));
    }

    // 5. 데이터베이스 연결 종료
    sqlite3_close(db);
    return 0;
}
```
<img width="1114" height="521" alt="image" src="https://github.com/user-attachments/assets/b3c82fe6-9d78-47f5-8732-25a2c6bc1b26" />
