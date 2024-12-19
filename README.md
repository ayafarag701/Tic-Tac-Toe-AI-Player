# Tic-Tac-Toe-AI-Player
import tkinter as tk
from tkinter import messagebox
import numpy as np
import time  # Import the time module
calls = 0  # Global variable to count algorithm calls
Tile = "-"
Player_X = "X"
Player_O = "O"
Current_Player = Player_O
board = [Tile] * 9
ai_technique = "Minimax"

def is_terminal(board):
    winning_combinations = [
        (0, 1, 2), (3, 4, 5), (6, 7, 8),
        (0, 3, 6), (1, 4, 7), (2, 5, 8),
        (0, 4, 8), (2, 4, 6)
    ]
    for a, b, c in winning_combinations:
        if board[a] == board[b] == board[c] and board[a] != Tile:
            return True, board[a]
    if all(cell != Tile for cell in board):
        return True, "tie"
    return False, None

def show_winner(winner):
    win_message = tk.Toplevel(root)
    win_message.title("Game Over")
    win_message.geometry("300x150")
    win_message.resizable(False, False)

    if winner == Player_X:
        bg_color = "#ff6f91"
        message = "AI wins! 😞"
    elif winner == Player_O:
        bg_color = "#4dff88"
        message = "You win! 🎉"
    elif winner == "tie":
        bg_color = "#ffd24d"
        message = "It's a tie! 😐"
    else:
        bg_color = "#d3d3d3"
        message = "Unknown result"

    win_message.configure(bg=bg_color)

    def on_close():
        win_message.destroy()
        reset_game()

    win_message.protocol("WM_DELETE_WINDOW", on_close)

    tk.Label(
        win_message,
        text=message,
        font=("Helvetica", 12, "bold"),
        bg=bg_color,
        fg="white"
    ).pack(pady=20)

    tk.Button(
        win_message,
        text="OK",
        font=("Helvetica", 10),
        bg="white",
        fg="black",
        command=lambda: [win_message.destroy(), reset_game()]
    ).pack(pady=10)

def evaluate_board(board):
    terminal = is_terminal(board)
    if terminal == Player_X:
        return 1
    elif terminal == Player_O:
        return -1
    else:
        return 0

def update_buttons():
    for i in range(9):
        buttons[i]["text"] = board[i]

def player_move(index):
    global Current_Player
    if board[index] == Tile:
        board[index] = Current_Player
        print(f"Player Move: {index}, Board: {board}")
        update_buttons()
        winner = is_terminal(board)[1]
        if winner:
            show_winner(winner)
        else:
            Current_Player = Player_X
            ai_move()

def ai_move():
    global Current_Player, calls  # Include called in the global variables
    start_time = time.time()  # Start timing

    if ai_technique == "Minimax":
        _, move = minimax(board, Player_X)
    elif ai_technique == "Alpha_Beta":
        _, move = minimax_with_alpha_beta(board, Player_X, float('-inf'), float('inf'))
    elif ai_technique == "Heuristic 1":
        _, move = minimax_with_heuristic1(board, 3, True, memo={})
    elif ai_technique == "Heuristic 2":
        _, move = minimax_with_heuristic2(board, 3, True, memo={})
    else:
        return

    end_time = time.time()  # End timing
    ai_time = end_time - start_time  # Calculate the time taken

    if move is not None:
        board[move] = Player_X
        print(f"AI Move: {move}, Board: {board}, Time taken: {ai_time:.6f} seconds, Calls: {calls}")
        calls = 0
        # Print AI time and calls
        update_buttons()
        winner = is_terminal(board)[1]
        if winner:
            show_winner(winner)
        Current_Player = Player_O

def change_ai_technique(technique):
    global ai_technique

    ai_technique = technique
    print(f"AI Technique Changed To: {ai_technique}")
    algorithm_label.config(text=f"Current Algorithm: {technique}")
    reset_game()

def get_symmetric_boards(board):
    board_matrix = [board[i:i + 3] for i in range(0, 9, 3)]
    symmetric_boards = []

    rotated_90 = list(zip(*board_matrix[::-1]))
    symmetric_boards.append([item for sublist in rotated_90 for item in sublist])

    rotated_180 = [row[::-1] for row in board_matrix[::-1]]
    symmetric_boards.append([item for sublist in rotated_180 for item in sublist])

    rotated_270 = list(zip(*board_matrix))[::-1]
    symmetric_boards.append([item for sublist in rotated_270 for item in sublist])

    mirrored_board = [row[::-1] for row in board_matrix]
    symmetric_boards.append([item for sublist in mirrored_board for item in sublist])

    return list(set(map(tuple, symmetric_boards)))

def minimax(board, player):
    global calls  # Declare the variable as global
    calls += 1  # Increment the call count

    value = is_terminal(board)[1]
    if value == Player_X:
        return 10, None
    elif value == Player_O:
        return -10, None
    elif value == "tie":
        return 0, None

    best_move = None
    if player == Player_X:
        best_score = float('-inf')
        for move in range(9):
            if board[move] == Tile:
                board[move] = Player_X
                score, _ = minimax(board, Player_O)
                board[move] = Tile
                if score > best_score:
                    best_score = score
                    best_move = move
        return best_score, best_move
    else:
        best_score = float('inf')
        for move in range(9):
            if board[move] == Tile:
                board[move] = Player_O
                score, _ = minimax(board, Player_X)
                board[move] = Tile
                if score < best_score:
                    best_score = score
                    best_move = move
        return best_score, best_move

def minimax_with_alpha_beta(board, player, alpha, beta):
    global calls  # Declare the variable as global
    calls += 1
    value = is_terminal(board)[1]
    if value == Player_X:
        return 10, None
    elif value == Player_O:
        return -10, None
    elif value == "tie":
        return 0, None

    best_move = None
    if player == Player_X:
        best_score = float('-inf')
        for move in range(9):
            if board[move] == Tile:
                board[move] = Player_X
                score, _ = minimax_with_alpha_beta(board, Player_O, alpha, beta)
                board[move] = Tile
                if score > best_score:
                    best_score = score
                    best_move = move
                alpha = max(alpha, best_score)
                if beta <= alpha:
                    break
        return best_score, best_move
    else:
        best_score = float('inf')
        for move in range(9):
            if board[move] == Tile:
                board[move] = Player_O
                score, _ = minimax_with_alpha_beta(board, Player_X, alpha, beta)
                board[move] = Tile
                if score < best_score:
                    best_score = score
                    best_move = move
                beta = min(beta, best_score)
                if beta <= alpha:
                    break
        return best_score, best_move

def evaluate_terminal_state(value, depth):
    if value == Player_X:
        return 10 - depth
    elif value == Player_O:
        return depth - 10
    else:
        return 0

def find_critical_moves(board, player):
    opponent = Player_X if player == Player_O else Player_O
    for poses in [[0, 1, 2], [3, 4, 5], [6, 7, 8], [0, 3, 6], [1, 4, 7], [2, 5, 8], [0, 4, 8], [2, 4, 6]]:
        values = [board[i] for i in poses]
        if values.count(player) == 2 and values.count(Tile) == 1:
            move = poses[values.index(Tile)]
            return [move]
        if values.count(opponent) == 2 and values.count(Tile) == 1:
            move = poses[values.index(Tile)]
            return [move]
    return [i for i in range(9) if board[i] == Tile]

def minimax_with_heuristic1(board, depth, is_maximizing_player, memo={}):
    terminal, value = is_terminal(board)

    if terminal:
        return evaluate_terminal_state(value, depth), None

    if depth == 0:
        return evaluate_board(board), None

    boards = min(tuple(board), *map(tuple, get_symmetric_boards(board)))
    if boards in memo:
        return memo[boards]

    if is_maximizing_player:
        max_eval = -float('inf')
        best_move = None
        available_moves = find_critical_moves(board, Player_X)
        for move in available_moves:
            board[move] = Player_X
            eval_score, _ = minimax_with_heuristic1(board, depth - 1, False, memo)
            board[move] = Tile
            if eval_score > max_eval:
                max_eval = eval_score
                best_move = move
        memo[boards] = (max_eval, best_move)
        return max_eval, best_move
    else:
        min_eval = float('inf')
        best_move = None
        available_moves = find_critical_moves(board, Player_O)
        for move in available_moves:
            board[move] = Player_O
            eval_score, _ = minimax_with_heuristic1(board, depth - 1, True, memo)
            board[move] = Tile
            if eval_score < min_eval:
                min_eval = eval_score
                best_move = move
        memo[boards] = (min_eval, best_move)
        return min_eval, best_move

def priority_moves_strategy(board, player):
    if board[4] == Tile:
        return [4]

    corners = [0, 2, 6, 8]
    available_corners = [i for i in corners if board[i] == Tile]
    if available_corners:
        return available_corners

    sides = [1, 3, 5, 7]
    available_sides = [i for i in sides if board[i] == Tile]
    if available_sides:
        return available_sides

    return [i for i in range(9) if board[i] == Tile]

def minimax_with_heuristic2(board, depth, is_maximizing_player, memo={}):
    terminal, value = is_terminal(board)

    if terminal:
        return evaluate_terminal_state(value, depth), None

    if depth == 0:
        return evaluate_board(board), None

    boards = min(tuple(board), *map(tuple, get_symmetric_boards(board)))

    if boards in memo:
        return memo[boards]

    if is_maximizing_player:
        max_eval = -float('inf')
        best_move = None
        available_moves = priority_moves_strategy(board, Player_X)
        for move in available_moves:
            board[move] = Player_X
            eval_score, _ = minimax_with_heuristic2(board, depth - 1, False, memo)
            board[move] = Tile
            if eval_score > max_eval:
                max_eval = eval_score
                best_move = move
        memo[boards] = (max_eval, best_move)
        return max_eval, best_move
    else:
        min_eval = float('inf')
        best_move = None
        available_moves = priority_moves_strategy(board, Player_O)
        for move in available_moves:
            board[move] = Player_O
            eval_score, _ = minimax_with_heuristic2(board, depth - 1, True, memo)
            board[move] = Tile
            if eval_score < min_eval:
                min_eval = eval_score
                best_move = move
        memo[boards] = (min_eval, best_move)
        return min_eval, best_move

root = tk.Tk()
root.title("Tic Tac Toe")
root.configure(bg="#F1C6D0")
root.geometry("1000x500")

board_x_start = 330
board_y_start = 100

buttons = []
for i in range(9):
    button = tk.Button(root, text=Tile, font=("Comic Sans MS", 24), height=2, width=5,
                       command=lambda i=i: player_move(i), bg="#8B008B", fg="white")
    x_pos = board_x_start + (i % 3) * 100
    y_pos = board_y_start + (i // 3) * 100
    button.place(x=x_pos, y=y_pos, width=80, height=80)
    buttons.append(button)

button_height = 60
start_y = 120

spacing = 100

minimax_button = tk.Button(root, text="Minimax", font=("Kristen ITC", 14),
                           command=lambda: change_ai_technique("Minimax"), bg="#8B008B", fg="white", bd=5)
minimax_button.place(x=800, y=start_y, width=150, height=button_height)

ab_button = tk.Button(root, text="Alpha-Beta", font=("Kristen ITC", 14),
                      command=lambda: change_ai_technique("Alpha_Beta"), bg="#8B008B", fg="white", bd=5)
ab_button.place(x=800, y=start_y + button_height, width=150, height=button_height)

h1_button = tk.Button(root, text="Heuristic 1", font=("Kristen ITC", 14),
                      command=lambda: change_ai_technique("Heuristic 1"), bg="#8B008B", fg="white", bd=5)
h1_button.place(x=800, y=start_y + 2 * button_height, width=150, height=button_height)

h2_button = tk.Button(root, text="Heuristic 2", font=("Kristen ITC", 14),
                      command=lambda: change_ai_technique("Heuristic 2"), bg="#8B008B", fg="white", bd=5)
h2_button.place(x=800, y=start_y + 3 * button_height, width=150, height=button_height)

algorithm_label = tk.Label(root, text="Current Algorithm: Minimax", font=("Kristen ITC", 16), fg="#FF7F32", bg="#F1C6D0")
algorithm_label.place(x=320, y=60)

reset_button = tk.Button(root, text="Reset", font=("Kristen ITC", 14), command=lambda: reset_game(), bg="#8B008B",
                         fg="white", bd=5)
reset_button.place(x=385, y=start_y + 5 * button_height, width=150, height=50)

exit_button = tk.Button(root, text="Exit", command=root.quit, font=("Kristen ITC", 12), fg="white", bg="#8B008B",
                        activebackground="#5A1E76", activeforeground="#5A1E76", borderwidth=0, highlightthickness=0,
                        bd=0, cursor="hand1")
exit_button.place(x=20, y=10, width=80, height=50)

label = tk.Label(root, text="Tic Tac Toe", font=("Kristen ITC", 24), fg="#8B008B", bg="#F1C6D0")
label.place(x=365, y=5)

def reset_game():
    global board, Current_Player , calls
    calls = 0
    board = [Tile] * 9
    Current_Player = Player_O
    update_buttons()

root.mainloop()
