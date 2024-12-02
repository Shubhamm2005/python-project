import pygame
import random
import sys

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 600, 750

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
GRAY = (169, 169, 169)
DARK_GRAY = (105, 105, 105)

# Initialize screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Save Boy X")

# Clock and FPS
clock = pygame.time.Clock()
FPS = 60

# Load customizable assets
background_images = [
    pygame.image.load("background1.jpeg"),  # Replace with your first background image
    pygame.image.load("background2.jpeg"),  # Replace with your second background image
    pygame.image.load("background3.jpeg"),  # Replace with your third background image
]
background_images = [pygame.transform.scale(img, (WIDTH, HEIGHT)) for img in background_images]
current_background_index = 0
background_change_interval = 300  # Change background every 300 frames

# Load player's car
CAR_WIDTH, CAR_HEIGHT = 50, 100
player_car_img = pygame.image.load("boy1.png")  # Replace with your player's car sprite
player_car_img = pygame.transform.scale(player_car_img, (CAR_WIDTH, CAR_HEIGHT))
player_x = WIDTH // 2 - CAR_WIDTH // 2
player_y = HEIGHT - CAR_HEIGHT - 20
player_speed = 7

# Load opponent's cars
OPPONENT_WIDTH, OPPONENT_HEIGHT = 50, 50
obstacle_images = [
    pygame.image.load("zombie.png"),  # Replace with your obstacle images
    pygame.image.load("zombie2.png"),  # Replace with your obstacle images
    pygame.image.load("zombie3.png")  # Replace with your obstacle images
]
obstacle_images = [pygame.transform.scale(img, (OPPONENT_WIDTH, OPPONENT_HEIGHT)) for img in obstacle_images]

# Obstacles (opponents)
obstacle_speed = 3
obstacles = []

# Power-ups
POWERUP_SIZE = 30
powerups = []
powerup_effects = {"boost": 1, "points": 1000}

# Score and game state
score = 0
font = pygame.font.Font(None, 10)
running = True
game_over = False

# Button dimensions
BUTTON_WIDTH, BUTTON_HEIGHT = 200, 50
restart_button = pygame.Rect(WIDTH // 2 - BUTTON_WIDTH // 2, HEIGHT // 2 + 50, BUTTON_WIDTH, BUTTON_HEIGHT)
exit_button = pygame.Rect(WIDTH // 2 - BUTTON_WIDTH // 2, HEIGHT // 2 + 120, BUTTON_WIDTH, BUTTON_HEIGHT)


# Functions
def create_obstacle():
    """Create an obstacle (opponent car) at a random x-coordinate."""
    x = random.randint(30, WIDTH - 30 - OPPONENT_WIDTH)
    y = -OPPONENT_HEIGHT
    image = random.choice(obstacle_images)  # Randomly select an obstacle image
    return {"rect": pygame.Rect(x, y, OPPONENT_WIDTH, OPPONENT_HEIGHT), "image": image}


def create_powerup():
    """Create a power-up at a random x-coordinate."""
    x = random.randint(50, WIDTH - 50 - POWERUP_SIZE)
    y = -POWERUP_SIZE
    return {"rect": pygame.Rect(x, y, POWERUP_SIZE, POWERUP_SIZE), "type": random.choice(["boost", "points"])}


def draw_text(text, color, x, y, size=36):
    """Draw text on the screen."""
    font = pygame.font.Font(None, size)
    render = font.render(text, True, color)
    screen.blit(render, (x, y))


def reset_game():
    """Reset the game state."""
    global player_x, player_y, obstacles, powerups, score, obstacle_speed, game_over
    player_x = WIDTH // 2 - CAR_WIDTH // 2
    player_y = HEIGHT - CAR_HEIGHT - 20
    obstacles = []
    powerups = []
    score = 0
    obstacle_speed = 5
    game_over = False


# Main Game Loop
frame_count = 0  # Initialize frame count

while running:
    if not game_over:
        # Game logic and rendering
        current_background_index = (frame_count // background_change_interval) % len(background_images)
        screen.blit(background_images[current_background_index], (0, 0))  # Draw background image

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        # Player controls
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player_x > 50:
            player_x -= player_speed
        if keys[pygame.K_RIGHT] and player_x < WIDTH - 50 - CAR_WIDTH:
            player_x += player_speed
        if keys[pygame.K_UP] and player_y > 0:
            player_y -= player_speed
        if keys[pygame.K_DOWN] and player_y < HEIGHT - CAR_HEIGHT:
            player_y += player_speed

        # Move and remove obstacles
        for obstacle in obstacles:
            obstacle["rect"].y += obstacle_speed
        obstacles = [obstacle for obstacle in obstacles if obstacle["rect"].y < HEIGHT]

        # Generate new obstacles
        if random.randint(1, 30) == 1:
            obstacles.append(create_obstacle())

        # Collision detection (obstacles)
        player_rect = pygame.Rect(player_x, player_y, CAR_WIDTH, CAR_HEIGHT)
        for obstacle in obstacles:
            if player_rect.colliderect(obstacle["rect"]):
                game_over = True  # End game on collision

        # Move and remove power-ups
        for powerup in powerups:
            powerup["rect"].y += obstacle_speed
        powerups = [powerup for powerup in powerups if powerup["rect"].y < HEIGHT]

        # Generate new power-ups
        if random.randint(1, 100) == 1:
            powerups.append(create_powerup())

        # Collision detection (power-ups)
        for powerup in powerups:
            if player_rect.colliderect(powerup["rect"]):
                powerup_type = powerup["type"]
                if powerup_type == "boost":
                    obstacle_speed += 2
                elif powerup_type == "points":
                    score += powerup_effects["points"]
                powerups.remove(powerup)

        # Draw player
        screen.blit(player_car_img, (player_x, player_y))

        # Draw obstacles (opponent cars)
        for obstacle in obstacles:
            screen.blit(obstacle["image"], (obstacle["rect"].x, obstacle["rect"].y))

        # Draw power-ups
        for powerup in powerups:
            color = GREEN if powerup["type"] == "boost" else BLUE
            pygame.draw.ellipse(screen, color, powerup["rect"])

        # Update score
        score += 1
        draw_text(f"Score: {score}", WHITE, 10, 10)

        # Increment frame count
        frame_count += 1

    else:
        # Game Over Screen
        screen.blit(background_images[current_background_index], (0, 0))  # Draw background image
        draw_text("Game Over", RED, WIDTH // 2 - 100, HEIGHT // 2 - 50, size=48)
        draw_text(f"Score: {score}", WHITE, WIDTH // 2 - 50, HEIGHT // 2, size=36)

        # Draw restart button
        pygame.draw.rect(screen, BLUE, restart_button)
        draw_text("Restart", WHITE, restart_button.x + 50, restart_button.y + 10, size=36)

        # Draw exit button
        pygame.draw.rect(screen, BLUE, exit_button)
        draw_text("Exit", WHITE, exit_button.x + 75, exit_button.y + 10, size=36)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = event.pos
                if restart_button.collidepoint(mouse_pos):
                    reset_game()
                if exit_button.collidepoint(mouse_pos):
                    running = False

    # Update display
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
sys.exit()
