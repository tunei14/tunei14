import pygame
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Ball Bounce Game")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
YELLOW = (255, 255, 0)
CYAN = (0, 255, 255)

# Game clock
clock = pygame.time.Clock()


# Block class
class Block:
    def __init__(self, x, y, width, height, color):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.color = color
        self.hit = False  # Track if the block has been hit

    def draw(self):
        if not self.hit:
            pygame.draw.rect(screen, self.color, (self.x, self.y, self.width, self.height))

    def collide(self, ball):
        # Check for collision with the ball
        if (self.x <= ball.x <= self.x + self.width) and (self.y <= ball.y <= self.y + self.height):
            self.hit = True
            return True
        return False


# Ball class
class Ball:
    def __init__(self, speed):
        self.x = WIDTH // 2
        self.y = HEIGHT // 2
        self.radius = 15
        self.x_speed = speed * random.choice([1, -1])
        self.y_speed = speed * random.choice([1, -1])

    def move(self):
        self.x += self.x_speed
        self.y += self.y_speed

    def bounce(self):
        if self.x <= 0 or self.x >= WIDTH - self.radius:
            self.x_speed *= -1
        if self.y <= 0:
            self.y_speed *= -1

    def reset(self):
        self.x = WIDTH // 2
        self.y = HEIGHT // 2
        self.x_speed *= random.choice([1, -1])
        self.y_speed = 5

    def draw(self):
        pygame.draw.circle(screen, CYAN, (self.x, self.y), self.radius)


# Paddle class
class Paddle:
    def __init__(self):
        self.width = 100
        self.height = 15
        self.x = (WIDTH // 2) - self.width // 2
        self.y = HEIGHT - self.height - 10
        self.speed = 10

    def move(self, dx):
        self.x += dx
        # Prevent paddle from going out of bounds
        if self.x < 0:
            self.x = 0
        if self.x > WIDTH - self.width:
            self.x = WIDTH - self.width

    def draw(self):
        pygame.draw.rect(screen, RED, (self.x, self.y, self.width, self.height))


# Create the start menu
def draw_start_menu():
    font = pygame.font.SysFont("Arial", 50)
    title_text = font.render("Ball Bounce Game", True, BLUE)
    start_text = font.render("Press ENTER to Start", True, GREEN)
    quit_text = font.render("Press Q to Quit", True, RED)

    screen.fill(WHITE)
    screen.blit(title_text, (WIDTH // 4, HEIGHT // 3))
    screen.blit(start_text, (WIDTH // 4, HEIGHT // 2))
    screen.blit(quit_text, (WIDTH // 4, HEIGHT // 1.5))

    pygame.display.flip()


# Create the game over menu
def draw_game_over_menu(score, level):
    font = pygame.font.SysFont("Arial", 50)
    game_over_text = font.render("Game Over!", True, RED)
    score_text = font.render(f"Score: {score}  Level: {level}", True, BLUE)
    restart_text = font.render("Press R to Restart", True, GREEN)
    quit_text = font.render("Press Q to Quit", True, RED)

    screen.fill(WHITE)
    screen.blit(game_over_text, (WIDTH // 3, HEIGHT // 3))
    screen.blit(score_text, (WIDTH // 4, HEIGHT // 2))
    screen.blit(restart_text, (WIDTH // 4, HEIGHT // 1.5))
    screen.blit(quit_text, (WIDTH // 4, HEIGHT // 1.2))

    pygame.display.flip()


# Create blocks
def create_blocks():
    blocks = []
    block_width = 80
    block_height = 30
    for row in range(4):  # 4 rows of blocks
        for col in range(10):  # 10 blocks per row
            x = col * (block_width + 5)
            y = row * (block_height + 5) + 50  # leave some space at the top
            color = random.choice([RED, GREEN, BLUE, YELLOW])
            blocks.append(Block(x, y, block_width, block_height, color))
    return blocks


# Main game loop
def game_loop():
    paddle = Paddle()
    ball = Ball(5)
    blocks = create_blocks()  # Create blocks
    score = 0
    level = 1
    running = True
    dx = 0  # Paddle horizontal movement (left-right)
    game_over = False

    while running:
        screen.fill(WHITE)

        if game_over:
            # Show Game Over menu
            draw_game_over_menu(score, level)
        else:
            # Draw blocks, paddle, and ball
            for block in blocks:
                block.draw()
            paddle.draw()
            ball.draw()

            # Handle events
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False

            # Handle keypresses for paddle movement
            keys = pygame.key.get_pressed()
            if keys[pygame.K_LEFT]:
                dx = -paddle.speed
            elif keys[pygame.K_RIGHT]:
                dx = paddle.speed
            else:
                dx = 0
            # Move paddle and ball
            paddle.move(dx)
            ball.move()
            ball.bounce()

            # Check for ball-paddle collision
            if ball.y + ball.radius >= paddle.y and paddle.x <= ball.x <= paddle.x + paddle.width:
                ball.y_speed *= -1  # Bounce the ball back
                score += 10  # Add points for bouncing the ball
                if score % 50 == 0:  # Increase speed every 50 points
                    level += 1
                    ball.x_speed += random.choice([-1, 1]) * 2
                    ball.y_speed += random.choice([-1, 1]) * 2

            # Check for ball-block collisions
            for block in blocks[:]:
                if block.collide(ball):
                    blocks.remove(block)  # Remove block on collision
                    ball.y_speed *= -1  # Bounce the ball
                    score += 20  # Add points for hitting a block
                    break  # Stop checking other blocks once a collision happens

            # If ball falls below paddle, game over
            if ball.y >= HEIGHT:
                game_over = True  # Set game over flag

            # Display the score and level
            font = pygame.font.SysFont("Arial", 30)
            score_text = font.render(f"Score: {score}  Level: {level}", True, BLACK)
            screen.blit(score_text, (10, 10))

            # Update the screen
            pygame.display.flip()

        # Handle game over actions
        if game_over:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_r:  # Restart the game
                        game_loop()
                    if event.key == pygame.K_q:  # Quit the game
                        running = False

        # Set frame rate (60 frames per second)
        clock.tick(60)

# Main menu loop
def main_menu():
    running = True
    while running:
        draw_start_menu()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:  # Start the game
                    game_loop()
                if event.key == pygame.K_q:  # Quit the game
                    clock.tick(60)
                    running = False

# Run the main menu
main_menu()

# Quit Pygame
pygame.quit()
