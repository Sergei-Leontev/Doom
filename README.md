from random import randint


class BoardException(Exception):
    pass

class BoardOutException(BoardException):
    def __str__(self):
        return "ВЫСТРЕЛ ВОСПРОИЗВЕДЕН МИМО ИГРОВОГО ПОЛЯ!!"

class BoardUsedException(BoardException):
    def __str__(self):
        return "ВЫСТРЕЛ УЖЕ БЫЛ ВОСПРОИЗВЕДЕН В ЭТУ КЛЕТКУ!!"

class BoardWrongShipException(BoardException):
    pass

class Dots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __repr__(self):
        return f"({self.x}, {self.y})"


class Ships:
    def __init__(self, bow, l, h):
        self.bow = bow
        self.l = l
        self.h = h
        self.lives = l

    @property
    def dots(self):
        ship_dots = []
        for i in range(self.l):
            cur_x = self.bow.x
            cur_y = self.bow.y

            if self.h == 0:
                cur_x += i

            elif self.h == 1:
                cur_y += i

            ship_dots.append(Dots(cur_x, cur_y))

        return ship_dots

    def shooten(self, shot):
        return shot in self.dots

class Field:
    def __init__(self, hid=False, size=6):
        self.size = size
        self.hid = hid

        self.count = 0

        self.field = [["O"] * size for _ in range(size)]

        self.busy = []
        self.ships = []

    def add_ship(self, ship):

        for d in ship.dots:
            if self.out(d) or d in self.busy:
                raise BoardWrongShipException()
        for d in ship.dots:
            self.field[d.x][d.y] = "■"
            self.busy.append(d)

        self.ships.append(ship)
        self.contour(ship)

    def contour(self, ship, verb=False):
        near = [
            (-1, -1), (-1, 0), (-1, 1),
            (0, -1), (0, 0), (0, 1),
            (1, -1), (1, 0), (1, 1)
        ]
        for d in ship.dots:
            for dx, dy in near:
                cur = Dots(d.x + dx, d.y + dy)
                if not (self.out(cur)) and cur not in self.busy:
                    if verb:
                        self.field[cur.x][cur.y] = "."
                    self.busy.append(cur)

    def __str__(self):
        res = ""
        res += "  | 1 | 2 | 3 | 4 | 5 | 6 |"
        for i, row in enumerate(self.field):
            res += f"\n{i + 1} | " + " | ".join(row) + " |"

        if self.hid:
            res = res.replace("■", "O")
        return res

    def out(self, d):
        return not ((0 <= d.x < self.size) and (0 <= d.y < self.size))

    def shot(self, d):
        if self.out(d):
            raise BoardOutException()

        if d in self.busy:
            raise BoardUsedException()

        self.busy.append(d)

        for ship in self.ships:
            if d in ship.dots:
                ship.lives -= 1
                self.field[d.x][d.y] = "X"
                if ship.lives == 0:
                    self.count += 1
                    self.contour(ship, verb=True)
                    print("КОАБЛЬ РАНЕН!")
                    return False
                else:
                    print("КРАБЛЬ ПОДБИТ!")
                    return True

        self.field[d.x][d.y] = "."
        print("Мимо!")
        return False

    def begin(self):
        self.busy = []


class Playr:
    def __init__(self, board, enemy):
        self.board = board
        self.enemy = enemy

    def ask(self):
        raise NotImplementedError()

    def move(self):
        while True:
            try:
                target = self.ask()
                repeat = self.enemy.shot(target)
                return repeat
            except BoardException as e:
                print(e)

class User(Playr):
    def ask(self):
        while True:
            cords = input("ХОД ИГРОКА: ").split()

            if len(cords) != 2:
                print(" ВВЕДИТЕ ДВЕ КООРДИНАТЫ !! ")
                continue

            x, y = cords

            if not (x.isdigit()) or not (y.isdigit()):
                print(" ВВЕСТИ ЧИСЛА! ")
                continue

            x, y = int(x), int(y)

            return Dots(x - 1, y - 1)
class Ai(Playr):
    def ask(self):
        d = Dots(randint(0, 5), randint(0, 5))
        print(f"ХОД Ai: {d.x + 1} {d.y + 1}")
        return d

class Game:
    def __init__(self, size=6):
        self.size = size
        pl = self.random_board()
        co = self.random_board()
        co.hid = True

        self.ai = Ai(co, pl)
        self.us = User(pl, co)

    def random_board(self):
        board = None
        while board is None:
            board = self.random_place()
        return board

    def random_place(self):
        lens = [3, 2, 2, 1, 1, 1, 1]
        board = Field(size=self.size)
        attempts = 0
        for l in lens:
            while True:
                attempts += 1
                if attempts > 2000:
                    return None
                ship = Ships(Dots(randint(0, self.size), randint(0, self.size)), l, randint(0, 1))
                try:
                    board.add_ship(ship)
                    break
                except BoardWrongShipException:
                    pass
        board.begin()
        return board

    def greet(self):
        print("<------------------->")
        print("|  ПРИВЕТСВУЕМ ВАС  |")
        print("|      В ИГРЕ       |")
        print("|    МЩРСКОЙ БОЙ    |")
        print("<------------------->")
        print("<------------------->")
        print("| ФОРМАТ ВВОДА: x y |")
        print("| x - НОМЕР СТРОКИ  |")
        print("| y - НОМЕР СТОЛБЦА |")
        print("<------------------->")

    def loop(self):
        num = 0
        while True:
            print("~|~" * 9)
            print("ДОСКА ИГРОКА:")
            print(self.us.board)
            print("~|~" * 9)
            print("~|~" * 9)
            print("ДОСКА Ai:")
            print(self.ai.board)
            print("~|~" * 9)
            if num % 2 == 0:
                print("/\\" * 10)
                print("ХОДИТ ИГРОК!")
                repeat = self.us.move()
                print("\/" * 10)
            else:
                print("/\\" * 20)
                print("ХОДИТ Ai!")
                repeat = self.ai.move()
                print("\/" * 20)
            if repeat:
                num -= 1

            if self.ai.board.count == 7:
                print("~|~" * 20)
                print("ИГРОК ВЫИГРАЛ!")
                print("~|~" * 20)
                break

            if self.us.board.count == 7:
                print("~|~" * 40)
                print("Ai ВЫИГРАЛ!")
                print("~|~" * 40)
                break
            num += 1

    def start(self):
        self.greet()
        self.loop()

g = Game()
g.start()
