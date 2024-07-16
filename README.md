    #include <iostream>
    #include <vector>
    #include <ctime>
    #include <conio.h>
    #include <windows.h>

    using namespace std;

    const int width = 10;
    const int height = 20;
    vector<vector<int>> board(height, vector<int>(width, 0));

    vector<vector<vector<int>>> tetrominoes = {
        {{1, 1, 1, 1}}, 
        {{1, 1, 1}, {0, 1, 0}}, 
        {{1, 1}, {1, 1}}, 
        {{1, 1, 0}, {0, 1, 1}}, 
        {{0, 1, 1}, {1, 1, 0}}, 
        {{1, 1, 1}, {1, 0, 0}}, 
        {{1, 1, 1}, {0, 0, 1}} 
    };

    struct Tetromino {
        vector<vector<int>> shape;
        int x, y;
    };

    Tetromino current;

    void initTetromino() {
        current.shape = tetrominoes[rand() % tetrominoes.size()];
        current.x = width / 2 - current.shape[0].size() / 2;
        current.y = 0;
    }

    bool checkCollision() {
        for (int i = 0; i < current.shape.size(); i++) {
            for (int j = 0; j < current.shape[i].size(); j++) {
                if (current.shape[i][j] && (current.y + i >= height || current.x + j < 0 || current.x + j >= width || board[current.y + i][current.x + j])) {
                    return true;
                }
            }
        }
        return false;
    }

    void placeTetromino() {
        for (int i = 0; i < current.shape.size(); i++) {
            for (int j = 0; j < current.shape[i].size(); j++) {
                if (current.shape[i][j]) {
                    board[current.y + i][current.x + j] = 1;
                }
            }
        }
    }

    void clearLines() {
        for (int i = 0; i < height; i++) {
            bool full = true;
            for (int j = 0; j < width; j++) {
                if (!board[i][j]) {
                    full = false;
                    break;
                }
            }
            if (full) {
                board.erase(board.begin() + i);
                board.insert(board.begin(), vector<int>(width, 0));
            }
        }
    }

    void rotateTetromino() {
        vector<vector<int>> rotated;
        for (int i = 0; i < current.shape[0].size(); i++) {
            vector<int> row;
            for (int j = current.shape.size() - 1; j >= 0; j--) {
                row.push_back(current.shape[j][i]);
            }
            rotated.push_back(row);
        }
        current.shape = rotated;
    }

    void draw(HANDLE hConsole) {
        COORD coord;
        coord.X = 0;
        coord.Y = 0;
        SetConsoleCursorPosition(hConsole, coord);
        for (int i = 0; i < height; i++) {
            for (int j = 0; j < width; j++) {
                if (board[i][j]) cout << "#";
                else {
                    bool isPartOfTetromino = false;
                    for (int m = 0; m < current.shape.size(); m++) {
                        for (int n = 0; n < current.shape[m].size(); n++) {
                            if (current.shape[m][n] && i == current.y + m && j == current.x + n) {
                                isPartOfTetromino = true;
                                break;
                            }
                        }
                    }
                    if (isPartOfTetromino) cout << "#";
                    else cout << ".";
                }
            }
            cout << endl;
        }
    }

    int main() {
        srand(time(0));
        initTetromino();

        HANDLE hConsole = GetStdHandle(STD_OUTPUT_HANDLE);

        DWORD lastUpdateTime = GetTickCount();
        DWORD lastDropTime = GetTickCount();
        const DWORD updateInterval = 50;
        const DWORD dropInterval = 500; 

        while (true) {
            // 檢查鍵盤輸入
            if (_kbhit()) {
                char key = _getch();
                switch (key) {
                    case 'a': current.x--; if (checkCollision()) current.x++; break;
                    case 'd': current.x++; if (checkCollision()) current.x--; break;
                    case 's': current.y++; if (checkCollision()) { current.y--; placeTetromino(); clearLines(); initTetromino(); } break;
                    case 'w': rotateTetromino(); if (checkCollision()) rotateTetromino(); rotateTetromino(); rotateTetromino(); break;
                }
            }

            // 更新遊戲邏輯
            DWORD currentTime = GetTickCount();
            if (currentTime - lastUpdateTime > updateInterval) {
                lastUpdateTime = currentTime;

                // 控制方塊的自動下降
                if (currentTime - lastDropTime > dropInterval) {
                    current.y++;
                    if (checkCollision()) {
                        current.y--;
                        placeTetromino();
                        clearLines();
                        initTetromino();
                        if (checkCollision()) {
                            cout << "Game Over!" << endl;
                            break;
                        }
                    }
                    lastDropTime = currentTime;
                }

                draw(hConsole);
            }

            Sleep(1); // 確保主循環不會佔用過多的 CPU 資源
        }
        return 0;
    }
