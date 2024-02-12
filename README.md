# Doom
play = [[" "] * 3 for i in range (3)]
play[0][0] = 'x'
play[0][2] = "o"
def min_play():
    print(f" 0 1 2")
    for i in range (3):
        print(f"{i} {play[i][0]} {play[i][1]}{play[i][2]}")
min_play()
