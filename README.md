run this python code in your preferred editor--

[it may take 1-2 minutes to run on low end computer... dont worry]







import pygame
import random
import tkinter as tk
from tkinter import simpledialog
import threading

# Constants
FPS = 30
CELL_SIZE = 20  # Each cell will be 20x20 pixels

# Directions for moving in the maze
DIRECTIONS = [(0, 1), (1, 0), (0, -1), (-1, 0)]  # Right, Down, Left, Up

# Maze generation using DFS
def generate_maze(rows, cols):
    maze = [[1 for _ in range(cols)] for _ in range(rows)]
    stack = []
    start_cell = (0, 0)
    maze[start_cell[0]][start_cell[1]] = 0
    stack.append(start_cell)

    while stack:
        current_cell = stack[-1]
        x, y = current_cell
        neighbors = []

        for dx, dy in DIRECTIONS:
            nx, ny = x + dx * 2, y + dy * 2
            if 0 <= nx < rows and 0 <= ny < cols and maze[nx][ny] == 1:
                neighbors.append((nx, ny))

        if neighbors:
            next_cell = random.choice(neighbors)
            nx, ny = next_cell
            maze[x + (nx - x) // 2][y + (ny - y) // 2] = 0
            maze[nx][ny] = 0
            stack.append(next_cell)
        else:
            stack.pop()

    return maze

# Set the exit to an open path
def place_exit(maze, rows, cols):
    exit_pos = [rows - 1, cols - 2]  # Set exit at bottom-right corner (near the bottom)
    maze[exit_pos[0]][exit_pos[1]] = 0  # Make sure the exit is an open space (path)
    return exit_pos

# Draw the maze
def draw_maze(screen, maze, player_pos, exit_pos, rows, cols):
    for i in range(rows):
        for j in range(cols):
            color = (255, 255, 255) if maze[i][j] == 0 else (0, 0, 0)
            pygame.draw.rect(screen, color, (j * CELL_SIZE, i * CELL_SIZE, CELL_SIZE, CELL_SIZE))

    # Draw the exit
    pygame.draw.rect(screen, (0, 255, 0), (exit_pos[1] * CELL_SIZE, exit_pos[0] * CELL_SIZE, CELL_SIZE, CELL_SIZE))
    
    # Draw player
    pygame.draw.rect(screen, (255, 0, 0), (player_pos[1] * CELL_SIZE, player_pos[0] * CELL_SIZE, CELL_SIZE, CELL_SIZE))

# Main game loop
def main(rows, cols):
    # Calculate screen dimensions based on grid size
    width, height = cols * CELL_SIZE, rows * CELL_SIZE
    
    pygame.init()
    screen = pygame.display.set_mode((width, height))
    pygame.display.set_caption("Maze Generator")
    clock = pygame.time.Clock()

    maze = generate_maze(rows, cols)
    exit_pos = place_exit(maze, rows, cols)  # Set the exit without generating a wall there
    player_pos = [0, 0]  # Starting position of the player

    running = True
    while running:
        screen.fill((0, 0, 0))
        draw_maze(screen, maze, player_pos, exit_pos, rows, cols)

        # Check for victory
        if player_pos == exit_pos:
            font = pygame.font.Font(None, 74)
            text = font.render("You Win!", True, (255, 255, 0))
            screen.blit(text, (width // 2 - text.get_width() // 2, height // 2 - text.get_height() // 2))
            pygame.display.flip()
            pygame.time.wait(2000)  # Wait for 2 seconds before quitting
            running = False

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_UP] and player_pos[0] > 0 and maze[player_pos[0] - 1][player_pos[1]] == 0:
            player_pos[0] -= 1
        if keys[pygame.K_DOWN] and player_pos[0] < rows - 1 and maze[player_pos[0] + 1][player_pos[1]] == 0:
            player_pos[0] += 1
        if keys[pygame.K_LEFT] and player_pos[1] > 0 and maze[player_pos[0]][player_pos[1] - 1] == 0:
            player_pos[1] -= 1
        if keys[pygame.K_RIGHT] and player_pos[1] < cols - 1 and maze[player_pos[0]][player_pos[1] + 1] == 0:
            player_pos[1] += 1

        pygame.display.flip()
        clock.tick(FPS)

    pygame.quit()

# Function to create the Tkinter prompt for grid size
def get_grid_size():
    root = tk.Tk()
    root.withdraw()  # Hide the root window

    # Prompt for grid size (rows and columns)
    rows = simpledialog.askinteger("Grid Size", "Enter the number of rows:", minvalue=5, maxvalue=30)
    cols = simpledialog.askinteger("Grid Size", "Enter the number of columns:", minvalue=5, maxvalue=30)

    # Return the grid size
    return rows, cols

# Function to show the loading screen
def show_loading_screen():
    loading_window = tk.Tk()
    loading_window.title("Loading")
    loading_window.geometry("300x100")

    label = tk.Label(loading_window, text="Loading... Please wait", font=("Helvetica", 14))
    label.pack(pady=20)

    return loading_window

# Function to start the game after loading
def start_game_after_loading(rows, cols):
    loading_window = show_loading_screen()

    # Start the game in a separate thread to avoid freezing the GUI
    threading.Thread(target=main, args=(rows, cols)).start()

    # Close the loading window once the game starts
    loading_window.after(1000, loading_window.destroy)

# Main execution
if __name__ == "__main__":
    rows, cols = get_grid_size()  # Get grid size from the user
    start_game_after_loading(rows, cols)  # Show loading screen and start the game
