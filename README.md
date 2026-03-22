# Bird-game
# Bird game with rectangle obstacles + sky background (no color issues)
import pygame
import random

pygame.init()

# Screen
width, height = 500, 700
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Bird Game - Simple Rectangles + Sky Background")

# Load sky background image
# Make sure you have 'sky.jpg' in same folder
try:
    sky = pygame.image.load("sky.jpg")
    sky = pygame.transform.scale(sky, (width, height))
except:
    sky = None  # if image not found, fallback

# Clock
clock = pygame.time.Clock()
FPS = 60

# Bird
bird_radius = 30
bird_x = 120
bird_y = 50  # bottom start
bird_vel = 0
gravity = 0.5
jump_strength = -12

# Obstacles (rectangles)
obs_width = 60
gap_size = 200
obstacle_gap = 300
obstacles = []
for i in range(2):
    x = width + i * obstacle_gap
    y = random.randint(200, height - 200)
    obstacles.append([x, y])

# Score & Level
score = 0
level = 1
font = pygame.font.SysFont(None, 40)

# Pause button
paused = False
pause_rect = pygame.Rect(width - 80, 10, 70, 40)

# Game loop
running = True
while running:
    # Draw sky background
    if sky:
        screen.blit(sky, (0,0))
    else:
        screen.fill((135, 206, 235))  # fallback light sky blue
    
    # Draw pause button
    pygame.draw.rect(screen, (200, 200, 200), pause_rect)
    pause_text = font.render("Pause", True, (0,0,0))
    screen.blit(pause_text, (width - 75, 15))
    
    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.MOUSEBUTTONDOWN:
            mouse_pos = pygame.mouse.get_pos()
            if pause_rect.collidepoint(mouse_pos):
                paused = not paused
            else:
                if not paused:
                    bird_vel = jump_strength
    
    if not paused:
        # Bird physics
        bird_vel += gravity
        bird_y += bird_vel
        if bird_y < bird_radius:
            bird_y = bird_radius
            bird_vel = 0
        if bird_y > height - bird_radius:
            bird_y = height - bird_radius
            bird_vel = 0
        
        # Draw bird (simple design)
        pygame.draw.circle(screen, (255,200,0), (bird_x, int(bird_y)), bird_radius)
        pygame.draw.polygon(screen, (255,150,0), [
            (bird_x - bird_radius//2, bird_y),
            (bird_x - bird_radius - 5, bird_y - 10),
            (bird_x - bird_radius - 5, bird_y + 10)
        ])
        pygame.draw.circle(screen, (0,0,0), (bird_x + 10, int(bird_y + 10)), 5)  # eye
        
        # Move and draw obstacles (simple rectangles)
        obs_speed = 5 + level - 1
        for obs in obstacles:
            obs[0] -= obs_speed
            
            # Top rectangle
            top_rect = pygame.Rect(obs[0]-obs_width//2, 0, obs_width, obs[1]-gap_size//2)
            pygame.draw.rect(screen, (100,50,0), top_rect)
            
            # Bottom rectangle
            bottom_rect = pygame.Rect(obs[0]-obs_width//2, obs[1]+gap_size//2, obs_width, height-(obs[1]+gap_size//2))
            pygame.draw.rect(screen, (100,50,0), bottom_rect)
            
            # Collision detection
            if top_rect.collidepoint(bird_x, bird_y) or bottom_rect.collidepoint(bird_x, bird_y):
                running = False
            
            # Recycle obstacle
            if obs[0] < -obs_width:
                obs[0] = width + obs_width
                obs[1] = random.randint(200, height - 200)
                score += 1
                if score % 5 == 0:
                    level += 1
        
        # Draw score and level
        score_text = font.render(f"Score: {score}", True, (0,0,0))
        screen.blit(score_text, (10, 10))
        level_text = font.render(f"Level: {level}", True, (0,0,0))
        screen.blit(level_text, (10, 50))
    else:
        paused_text = font.render("Game Paused", True, (0,0,0))
        screen.blit(paused_text, (width//2 - 100, height//2 - 20))
    
    pygame.display.update()
    clock.tick(FPS)

pygame.quit()