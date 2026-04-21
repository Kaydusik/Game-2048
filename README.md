[game2048.py](https://github.com/user-attachments/files/26948698/game2048.py)
# Начало кода, import необходим для библиотеки и объявляю переменную в tk (название)
import tkinter as tk
from tkinter import messagebox
import random

# Объявляю главный класс программы (объект)
class Game2048:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("2048 Game")
        self.root.geometry("500x650")
        self.root.bind("<Key>", self.key_press) # Объясление слушателя нажатия клавиш.
        
        # Цвета для плиток
        self.colors = {
            0: ("#cdc1b4", "#000000"),
            2: ("#fce5d0", "#000000"),
            4: ("#f8dda9", "#000000"),
            8: ("#f2b179", "#000000"),
            16: ("#f58a51", "#000000"),
            32: ("#fa6843", "#000000"),
            64: ("#fa4c25", "#000000"),
            128: ("#f792f8", "#000000"),
            256: ("#fe71dd", "#000000"),
            512: ("#ff47f9", "#000000"),
            1024: ("#7767ef", "#000000"),
            2048: ("#4535f6", "#000000")
        }
        
        # Игровой счет (self.score и best_score) которые суммирует числа, grid сохряняет ячейки что находится в данный момент, tile_labels я сохраняю ссылку на ячейку, size размер таблицы, tiles сами ячейки и применяем цвет ячейки.
        self.score = 0
        self.best_score = 0
        self.grid = [[0]*4 for _ in range(4)]
        self.tile_labels = [[None]*4 for _ in range(4)]
        self.tiles = [[None]*4 for _ in range(4)]
        self.sizes = 100
        self.setup_ui()
        self.new_game()
        
        # Вызов функции на 37 строке, а сама функция 41 строка
    def setup_ui(self):
        # Заголовок
        title = tk.Label(self.root, text="2048", font=("Arial", 40, "bold"), 
                        bg="#bbada0", fg="#776e65")
        title.pack(pady=10)
        
        # Счет
        self.score_label = tk.Label(self.root, text="Счет: 0", font=("Arial", 16, "bold"),
                                   bg="#bbada0", fg="white")
        self.score_label.pack()
        
        self.best_label = tk.Label(self.root, text="Рекорд: 0", font=("Arial", 16, "bold"),
                                  bg="#bbada0", fg="white")
        self.best_label.pack()
        
        # Кнопка новой игры
        new_game_btn = tk.Button(self.root, text="Новая игра", font=("Arial", 14),
                               command=self.new_game, bg="#8f7a66", fg="white")
        new_game_btn.pack(pady=5)
        
        # Игровое поле, и какое будет поле в целом через width и height. Canvas это основное игровое поле. И вызвать функцию pack на 67 строке.
        self.game_frame = tk.Canvas(
            self.root,
            width= 4 * self.sizes,
            height= 4 * self.sizes
        )
        self.game_frame.pack()
        
        # Создаем сетку 4x4, (x1,y1) это левый верхний угол ячейки, а (x2,y2) это правый нижний угол по диогонали.
        for i in range(4):
            for j in range(4):
                x1 = i * self.sizes
                y1 = j * self.sizes
                x2 = x1 + self.sizes
                y2 = y1 + self.sizes

                # Рисуем прямоугольник (ячейку) и добовляем цветоколор.
                rect_id = self.game_frame.create_rectangle(
                    x1, y1, x2, y2,
                    fill="#cdc1b4",
                    outline="#000000"
                )
                self.tiles[j][i] = rect_id # Сохраняю ячейку чтобы менять ей цвет.

                # Добавляем текст внутри ячейки, центрируем. 
                text_id = self.game_frame.create_text(
                    x1 + self.sizes // 2,
                    y1 + self.sizes // 2,
                    text="",
                    font=("Arial", 24, "bold")
                )
                self.tile_labels[j][i] = text_id # Вместо ячейки, сохраняю текст.
                
    # Начало игры, обнуление и перезапуск.
    def new_game(self):
        self.grid = [[0]*4 for _ in range(4)] # grid это числа в ячейках.  
        self.score = 0
        self.update_score()
        self.add_tile() # Один вызов этих функции, равен заполнению случайной ячейки случайным числом (2 или 4). 
        self.add_tile()
        self.update_board()
    
    def add_tile(self):
        empty = [(i,j) for i in range(4) for j in range(4) if self.grid[i][j] == 0] # Проходит по всему массиву и выбирает только пустые в объект empty. 
        if empty:
            i, j = random.choice(empty) # Случайно выбирает 2 цифры, индекс строки и индекс столбца. 
            self.grid[i][j] = random.choice([2, 4] * 9 + [4]) # Создает массив чисел потом выбирает одно случайно. 
    
    def update_board(self): # Она просто меняет ячейку цвет или текст. И вызывается функция root.update. 
        for i in range(4):
            for j in range(4):
                num = self.grid[i][j]
                label = self.tile_labels[i][j]
                tile = self.tiles[i][j]
                bg, fg = self.colors[num]
                self.game_frame.itemconfig(label, text=str(num) if num > 0 else "")
                self.game_frame.itemconfig(tile, fill=bg)
        self.root.update()
    
    def left(self):
        moved = False
        for i in range(4):
            prev = self.grid[i][:] # i он делает строку : копирует всю строку под индексом i.
            self.grid[i] = self.merge_row(self.grid[i]) # Объединяют все в 0 индекс.
            if self.grid[i] != prev:
                moved = True
        return moved
    
    def right(self):
        moved = False
        for i in range(4):
            prev = self.grid[i][:] # i он делает строку : копирует всю строку под индексом i.
            self.grid[i] = self.grid[i][::-1] # Переворачивет строку с лево направо, с направо, налево. 
            self.grid[i] = self.merge_row(self.grid[i]) # Объединяют все в 0 индекс.
            self.grid[i] = self.grid[i][::-1]
            if self.grid[i] != prev: # Проверка на поменянный массив строки. 
                moved = True
        return moved
    
    def up(self):
        moved = False
        for j in range(4):
            prev = [self.grid[i][j] for i in range(4)] 
            col = self.merge_col(j) 
            for i in range(4):
                self.grid[i][j] = col[i]
            if col != prev:
                moved = True
        return moved
    
    def down(self):
        moved = False
        for j in range(4):
            prev = [self.grid[i][j] for i in range(4)]
            col = [self.grid[i][j] for i in range(4)][::-1]
            col = self.merge_col(j) # Объединение столбца.
            col = col[::-1]
            for i in range(4):
                self.grid[i][j] = col[i]
            if col != prev:
                moved = True
        return moved
    
    def merge_row(self, row):
        # Убираем нули
        new = [x for x in row if x != 0]
        # Сливаем
        i = 0
        while i < len(new) - 1:
            if new[i] == new[i+1]:
                new[i] *= 2
                self.score += new[i]
                new.pop(i+1)
            i += 1
        # Добавляем нули
        new += [0] * (4 - len(new))
        return new
    
    def merge_col(self, j):
        col = [self.grid[i][j] for i in range(4)]
        return self.merge_row(col) # Объединение столбца.
    
    def check_win(self):
        return 2048 in [x for row in self.grid for x in row]
    
    def check_game_over(self):
        if any(0 in row for row in self.grid):
            return False
        for row in self.grid:
            for i in range(3):
                if row[i] == row[i+1]:
                    return False
        for j in range(4):
            for i in range(3):
                if self.grid[i][j] == self.grid[i+1][j]:
                    return False
        return True
    
    def update_score(self): # Полный перезапуск игры и лучший счет обновляется визуально. 
        self.score_label.config(text=f"Счет: {self.score}")
        self.best_score = max(self.best_score, self.score)
        self.best_label.config(text=f"Рекорд: {self.best_score}")
    
    def key_press(self, event):
        moved = False # Флаг выполнения действия. 
        key = event.keysym.lower() # Кнопка на которую нажали.
        
        if key == "a": # Проверка на какую клавишу нажал игрок.
            moved = self.left() # Игрок нажала на (a) их нужно сдвинуть налево. Возвращает True или False в зависимости смогла ли функция сдвинуть ячейки.
        elif key == "d":
            moved = self.right()
        elif key == "w":
            moved = self.up()
        elif key == "s":
            moved = self.down()
        
        if moved: # Проверяет ячейку при нажатии.
            self.add_tile()
            self.update_score()
            self.update_board()
            
            if self.check_win():
                messagebox.showinfo("Победа!", "Вы достигли 2048! Продолжить?")
            elif self.check_game_over():
                messagebox.showinfo("Игра окончена!", f"Итоговый счет: {self.score}")
    
    def run(self):
        self.root.mainloop()

# Запуск игры
if __name__ == "__main__":
    game = Game2048()
    game.run()
