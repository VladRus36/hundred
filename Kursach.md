#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <locale.h>
#define MAX_SIZE 5
 
int printField(int** field, int size);
void printMenu();
int playing(int size, int level);
void printrules();
char hint(char* fname);
int saveGame(int size, int** field);
int displayRes(int size, int** field);

int main() {
    setlocale(0, "");
    int field[MAX_SIZE][MAX_SIZE];
    char stop = '@';
    int size = 3, level = 1, row, col, cont, choice = 0;
    const char* fname = "hint.txt";
    puts("Добро пожаловать в игру 'СТО'. Желаем успешного прохождения этой увлекательной головоломки\n");
    while (stop != '$') {
        printMenu();
        scanf("%d", &choice);
        switch (choice) {
        case 1: {
            playing(level, size);
            break;
        }
        case 2: {
            printrules();
            break;
        }
        case 3: {
            puts("\n\n");
            puts("Перед вами пример исходного и победного полей\n");
            hint(fname);
            puts("\n");
            break;
        }
        case 4: {
            puts("\nРезультаты прошлых игр: ");
            displayRes(size, field);
            break;
        }
        case 5: {
            stop = '$';
            break;
        }
        default:
            puts("Неверный выбор опции!\n");
            continue;
        }
    } puts("\nДосвидания!");
    return 0;
}
int printField(int** field, int size) {
    printf("\n  ");
    for (int i = 0; i < size; i++) {
        printf("%2d ", i + 1);
    }
    printf("\n");
    for (int i = 0; i < size; i++) {
        printf("----");
    }
    printf("\n");
    for (int i = 0; i < size; i++) {
        printf("%d|", i + 1);
        for (int j = 0; j < size; j++) {
            printf("%2d|", field[i][j]);
        }
        printf("\n");
        for (int j = 0; j < size; j++) {
            printf("----");
        }
        printf("\n");
    }
}

void printMenu()
{
    puts("Игровое меню");
    puts("1 - Выбрать уровень");
    puts("2 - Об игре");
    puts("3 - Посмотреть пример верно заполненного поля");
    puts("4 - Загрузить результаты прошлых игр");
    puts("5 - Покинуть игру");
    return 1;
}


int playing(int size, int level)
{
    int rowSum = 0;
    int colSum = 0;
    do {
        puts("Выберите уровень сложности: ");
        puts("1 - простой (поле 3х3)");
        puts("2 - средний (поле 4х4)");
        puts("3 - сложный (поле 5х5)");
        scanf("%d", &level);
        switch (level) {
        case 1:
            size = 3;
            break;
        case 2:
            size = 4;
            break;
        case 3:
            size = 5;
            break;
        default:
            puts("Некорректный ввод");
            continue;
        }
        int** field = (int**)malloc(size * sizeof(int*));
        for (int i = 0; i < size; i++) {
            field[i] = (int*)malloc(size * sizeof(int));
        }
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                field[i][j] = rand() % 9 + 1;
            }
        }
        printf("Начальное игровое поле:\n");
        printField(field, size);
        int row, col, num, cont = 1;
        do {
            puts("Введите номер строки, столбца и цифру(через пробел): ");
            scanf("%d %d %d", &row, &col, &num);
            if (row > size || col > size || level > 3 || num > 9 || num < 0 || row < 1 || col < 1) {
                puts("Некорректный ввод! Будте внимательнее!\n");
                continue;
            }
            row--;
            col--;
            field[row][col] = field[row][col] * 10 + num;
            printField(field, size);
            puts("Хотите добавить ещё цифру?");
            puts("Да - 1; нет - 0");
            scanf("%d", &cont);
        } while (cont != 0);
        int rowSum = 0, colSum = 0;
        int choice;
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                rowSum += field[i][j];
                colSum += field[j][i];
            }
        }
        for (int i = 0; i < size; i++) {
            rowSum = 0;
            colSum = 0;
            for (int j = 0; j < size; j++) {
                rowSum += field[i][j];
                colSum += field[j][i];
            }
            if (rowSum != 100 || colSum != 100) {
                printf("Попробуйте снова!\n");
                saveGame(size, field);
                return;
            }
        }
        printf("Поздравляю, вы выиграли!\n");
        saveGame(size, level, field);
        for (int i = 0; i < size; i++) {
            free(field[i]);
        }
        free(field);
        return;
    } while (rowSum != 100 && colSum != 100);
}


void printrules() 
{
    puts("\n\nПравила игры:\n");
    puts("Вам необходимо дополнить поле недостающими цифрами, чтобы сумма каждой строки и столбца была равна 100.\n");
    puts("Вы можете вводить в любую ячейку только одну дополнительную цифру, чтобы число в ячейке было максимум двухзначным, иначе игра теряет смысл.\n");
    puts("Вы можете посмотреть на пример верно заполненного поля. Для этого вам необходимо выбрать соответсвующую опцию игрового меню.\n");
    puts("Удачной игры!\n");
    return 0;
}

char hint(char* fname)
{
    FILE* out;
    out = fopen(fname, "r");
    int c;
    while ((c = fgetc(out)) != EOF) {
        putchar(c);
    }
    fclose(out);
}

int saveGame(int size, int** field) {
    FILE* file = fopen("results.txt", "a");
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            fprintf(file, "%d ", field[i][j]);
        }
        fprintf(file, "\n");
    }
    fprintf(file, "\n");
    fclose(file);
}

int displayRes(int size, int** field)
{
    FILE* input;
    input = fopen("results.txt", "r");
    if (input == NULL) {
        puts("Файл пуст!");
        fclose(input);
    }
    int c;
    while ((c = fgetc(input)) != EOF) {
        putchar(c);
    }
    fclose(input);
    return 0;
}
