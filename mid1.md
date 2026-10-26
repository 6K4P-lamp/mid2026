
### 문제
\[mid1.md\] 무지개 블록깨기 게임을 C언어로 작성하시오. (그래픽 라이브러리 활용)

### 프롬프트
raylib를 사용하여 무지개 블록깨기 게임 코드를 작성해줘.<br>
한글은 사용하지 않아야 하고, C 언어로만 구현해야 해.

### 답
```c
#include "raylib.h"
#include <stdbool.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define ROWS 6
#define COLS 10
#define BLOCK_WIDTH 70
#define BLOCK_HEIGHT 20
#define BLOCK_PADDING 10
#define OFFSET_X 45
#define OFFSET_Y 60

typedef struct {
    Rectangle rect;
    bool active;
    Color color;
} Block;

typedef struct {
    Vector2 position;
    Vector2 speed;
    float radius;
} Ball;

typedef struct {
    Rectangle rect;
    float speed;
} Paddle;

int main(void) {
    // Initialize window and context
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Breakout Game");
    SetTargetFPS(60);

    // Rainbow color palette for the block rows
    Color rainbowColors[ROWS] = {
        RED,
        ORANGE,
        YELLOW,
        GREEN,
        BLUE,
        PURPLE
    };

    // Initialize blocks
    Block blocks[ROWS][COLS];
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            blocks[i][j].rect = (Rectangle){
                OFFSET_X + j * (BLOCK_WIDTH + BLOCK_PADDING),
                OFFSET_Y + i * (BLOCK_HEIGHT + BLOCK_PADDING),
                BLOCK_WIDTH,
                BLOCK_HEIGHT
            };
            blocks[i][j].active = true;
            blocks[i][j].color = rainbowColors[i];
        }
    }

    // Initialize paddle
    Paddle paddle = {
        { SCREEN_WIDTH / 2.0f - 50.0f, SCREEN_HEIGHT - 50.0f, 100.0f, 20.0f },
        8.0f
    };

    // Initialize ball
    Ball ball = {
        { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f },
        { 5.0f, -5.0f },
        8.0f
    };

    int lives = 3;
    int score = 0;
    bool gameOver = false;
    bool victory = false;

    // Main game loop
    while (!WindowShouldClose()) {
        if (!gameOver && !victory) {
            // Paddle movement
            if (IsKeyDown(KEY_LEFT)) paddle.rect.x -= paddle.speed;
            if (IsKeyDown(KEY_RIGHT)) paddle.rect.x += paddle.speed;

            // Screen boundary constraints for paddle
            if (paddle.rect.x < 0) paddle.rect.x = 0;
            if (paddle.rect.x + paddle.rect.width > SCREEN_WIDTH) {
                paddle.rect.x = SCREEN_WIDTH - paddle.rect.width;
            }

            // Ball movement
            ball.position.x += ball.speed.x;
            ball.position.y += ball.speed.y;

            // Wall collisions (left and right)
            if (ball.position.x - ball.radius < 0) {
                ball.position.x = ball.radius;
                ball.speed.x *= -1.0f;
            }
            if (ball.position.x + ball.radius > SCREEN_WIDTH) {
                ball.position.x = SCREEN_WIDTH - ball.radius;
                ball.speed.x *= -1.0f;
            }

            // Wall collision (top)
            if (ball.position.y - ball.radius < 0) {
                ball.position.y = ball.radius;
                ball.speed.y *= -1.0f;
            }

            // Bottom collision (lose a life)
            if (ball.position.y + ball.radius > SCREEN_HEIGHT) {
                lives--;
                if (lives <= 0) {
                    gameOver = true;
                }
                else {
                    ball.position = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
                    ball.speed = (Vector2){ 5.0f, -5.0f };
                }
            }

            // Paddle collision
            if (CheckCollisionCircleRec(ball.position, ball.radius, paddle.rect)) {
                ball.speed.y *= -1.0f;
                // Adjust horizontal bounce angle depending on where the ball hits the paddle
                float hitFactor = (ball.position.x - (paddle.rect.x + paddle.rect.width / 2.0f)) / (paddle.rect.width / 2.0f);
                ball.speed.x = hitFactor * 6.0f;
            }

            // Block collisions
            bool allCleared = true;
            for (int i = 0; i < ROWS; i++) {
                for (int j = 0; j < COLS; j++) {
                    if (blocks[i][j].active) {
                        allCleared = false;
                        if (CheckCollisionCircleRec(ball.position, ball.radius, blocks[i][j].rect)) {
                            blocks[i][j].active = false;
                            ball.speed.y *= -1.0f;
                            score += 10;
                        }
                    }
                }
            }

            if (allCleared) {
                victory = true;
            }
        }
        else {
            // Restart game on pressing SPACE
            if (IsKeyPressed(KEY_SPACE)) {
                gameOver = false;
                victory = false;
                lives = 3;
                score = 0;
                ball.position = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
                ball.speed = (Vector2){ 5.0f, -5.0f };
                for (int i = 0; i < ROWS; i++) {
                    for (int j = 0; j < COLS; j++) {
                        blocks[i][j].active = true;
                    }
                }
            }
        }

        // Drawing
        BeginDrawing();
        ClearBackground(RAYWHITE);

        // Draw active rainbow blocks
        for (int i = 0; i < ROWS; i++) {
            for (int j = 0; j < COLS; j++) {
                if (blocks[i][j].active) {
                    DrawRectangleRec(blocks[i][j].rect, blocks[i][j].color);
                    DrawRectangleLinesEx(blocks[i][j].rect, 1, BLACK);
                }
            }
        }

        // Draw paddle and ball
        DrawRectangleRec(paddle.rect, DARKGRAY);
        DrawCircleV(ball.position, ball.radius, MAROON);

        // Draw score and lives UI
        DrawText(TextFormat("Score: %d", score), 20, 20, 20, DARKGRAY);
        DrawText(TextFormat("Lives: %d", lives), SCREEN_WIDTH - 120, 20, 20, DARKGRAY);

        // Game Over or Victory screens
        if (gameOver) {
            DrawText("GAME OVER", SCREEN_WIDTH / 2 - MeasureText("GAME OVER", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, RED);
            DrawText("Press SPACE to Restart", SCREEN_WIDTH / 2 - MeasureText("Press SPACE to Restart", 20) / 2, SCREEN_HEIGHT / 2 + 10, 20, DARKGRAY);
        }
        else if (victory) {
            DrawText("VICTORY!", SCREEN_WIDTH / 2 - MeasureText("VICTORY!", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, GREEN);
            DrawText("Press SPACE to Restart", SCREEN_WIDTH / 2 - MeasureText("Press SPACE to Restart", 20) / 2, SCREEN_HEIGHT / 2 + 10, 20, DARKGRAY);
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

<img width="794" height="621" alt="image" src="https://github.com/user-attachments/assets/8dbb37f1-78a0-49a2-9ce1-bb9f6b81613e" />
