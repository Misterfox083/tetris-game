import pygame
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
SCREEN_WIDTH = 300
SCREEN_HEIGHT = 600
BLOCK_SIZE = 30

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (128, 128, 128)
COLORS = [
    (0, 255, 255),  # Cyan (I)
    (255, 255, 0),  # Yellow (O)
    (255, 165, 0),  # Orange (L)
    (0, 0, 255),    # Blue (J)
    (0, 255, 0),    # Green (S)
    (255, 0, 0),    # Red (Z)
    (128, 0, 128)   # Purple (T)
]

# Tetromino shapes
SHAPES = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[1, 1, 1], [1, 0, 0]],  # L
    [[1, 1, 1], [0, 0, 1]],  # J
    [[0, 1, 1], [1, 1, 0]],  # S
    [[1, 1, 0], [0, 1, 1]],  # Z
    [[1, 1, 1], [0, 1, 0]]   # T
]

# Initialize screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Tetris")

# Clock for controlling game speed
clock = pygame.time.Clock()

# Game variables
grid = [[0 for _ in range(SCREEN_WIDTH // BLOCK_SIZE)] for _ in range(SCREEN_HEIGHT // BLOCK_SIZE)]
current_piece = None
current_x = SCREEN_WIDTH // BLOCK_SIZE // 2
current_y = 0
score = 0
game_over = False

# Function to create a new piece
def new_piece():
    global current_piece, current_x, current_y
    shape = random.choice(SHAPES)
    color = COLORS[SHAPES.index(shape)]  # Correct color assignment
    current_piece = {"shape": shape, "color": color}
    current_x = SCREEN_WIDTH // BLOCK_SIZE // 2 - len(shape[0]) // 2
    current_y = 0
    if not valid_position(current_piece["shape"], (current_x, current_y)):
        return False
    return True

# Function to check if a position is valid
def valid_position(piece, offset):
    off_x, off_y = offset
    for y, row in enumerate(piece):
        for x, cell in enumerate(row):
            if cell:
                if x + off_x < 0 or x + off_x >= len(grid[0]) or y + off_y >= len(grid) or grid[y + off_y][x + off_x]:
                    return False
    return True

# Function to rotate a piece
def rotate(piece):
    return [list(row) for row in zip(*piece[::-1])]

# Function to clear lines and update score
def clear_lines():
    global grid, score
    lines_cleared = 0
    for y in range(len(grid)):
        if all(grid[y]):  # Check if the entire row is filled
            del grid[y]
            grid.insert(0, [0 for _ in range(len(grid[0]))])
            lines_cleared += 1
    # Update score based on lines cleared
    if lines_cleared == 1:
        score += 100
    elif lines_cleared == 2:
        score += 300
    elif lines_cleared == 3:
        score += 500
    elif lines_cleared == 4:
        score += 800

# Function to draw the grid
def draw_grid():
    for y in range(len(grid)):
        for x in range(len(grid[y])):
            if grid[y][x]:
                pygame.draw.rect(screen, grid[y][x], (x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))
            pygame.draw.rect(screen, GRAY, (x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE), 1)

# Main game loop
new_piece()
while not game_over:
    screen.fill(BLACK)

    # Handle events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            game_over = True
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                if valid_position(current_piece["shape"], (current_x - 1, current_y)):
                    current_x -= 1
            if event.key == pygame.K_RIGHT:
                if valid_position(current_piece["shape"], (current_x + 1, current_y)):
                    current_x += 1
            if event.key == pygame.K_DOWN:
                if valid_position(current_piece["shape"], (current_x, current_y + 1)):
                    current_y += 1
            if event.key == pygame.K_SPACE:
                rotated = rotate(current_piece["shape"])
                if valid_position(rotated, (current_x, current_y)):
                    current_piece["shape"] = rotated

    # Move piece down
    if valid_position(current_piece["shape"], (current_x, current_y + 1)):
        current_y += 1
    else:
        for y, row in enumerate(current_piece["shape"]):
            for x, cell in enumerate(row):
                if cell:
                    grid[current_y + y][current_x + x] = current_piece["color"]
        clear_lines()
        if not new_piece():
            game_over = True

    # Draw grid
    draw_grid()

    # Draw current piece
    for y, row in enumerate(current_piece["shape"]):
        for x, cell in enumerate(row):
            if cell:
                pygame.draw.rect(screen, current_piece["color"], ((current_x + x) * BLOCK_SIZE, (current_y + y) * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE))

    # Draw score
    font = pygame.font.SysFont("comicsans", 30)
    text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(text, (10, 10))

    # Update display
    pygame.display.update()
    clock.tick(5)

# Game over screen
font = pygame.font.SysFont("comicsans", 50)
text = font.render("Game Over", True, WHITE)
screen.blit(text, (SCREEN_WIDTH // 2 - text.get_width() // 2, SCREEN_HEIGHT // 2 - text.get_height() // 2))
pygame.display.update()
pygame.time.wait(3000)

# Quit Pygame
pygame.quit()
