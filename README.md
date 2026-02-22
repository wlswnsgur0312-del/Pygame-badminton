# 🏸 Python Badminton Game Project

### 📝 Project Description
This project simulates a **2D Badminton Match** developed using **Python**.
It features realistic shuttlecock physics (projectile motion with air resistance) and player movement mechanics.
(파이썬을 활용해 개발한 2D 배드민턴 게임입니다. 셔틀콕의 포물선 운동과 공기 저항 물리 효과를 구현했습니다.)

### 🛠️ Tech Stack
* **Language:** Python 3.14
* **Library:** Pygame
* **Key Features:**
    * **Shuttlecock Physics:** Implemented gravity and deceleration
    * **Player Control:** Smooth racket movement & smash mechanics
    * **Scoring System:** Real-time score tracking (Match Point logic)

### 📸 Screenshot
<img width="1590" height="1067" alt="image" src="https://github.com/user-attachments/assets/2c70d7a6-a8a6-403c-968c-2e4bfe22894b" />


### 🚀 How to Run
1. Install Python 3.x
2. Install Pygame: `pip install pygame`
3. Run the game: `python main.py` (or your file name)
# Pygame-badminton[badminton.py](https://github.com/user-attachments/files/25467053/badminton.py)
import pygame
import sys

# --- 초기 설정 ---
pygame.init()

WIDTH, HEIGHT = 800, 500
SCREEN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Badminton 2P: Wall Bounce & Smash")

# 색상 정의
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (50, 100, 255)   # P1
RED = (255, 50, 50)     # P2
YELLOW = (255, 255, 0) 
NET_COLOR = (100, 100, 100)
GREEN = (0, 255, 0)

clock = pygame.time.Clock()
FPS = 60

# --- 게임 변수 ---
GRAVITY = 0.4
JUMP_STRENGTH = -12
MOVE_SPEED = 7
SWING_DURATION = 15 
AIR_RESISTANCE = 0.985 

# 객체 생성
player1 = pygame.Rect(100, HEIGHT - 100, 30, 80)
player2 = pygame.Rect(WIDTH - 130, HEIGHT - 100, 30, 80)
net = pygame.Rect(WIDTH // 2 - 2, HEIGHT - 150, 4, 150)
ball = pygame.Rect(WIDTH // 2, 100, 15, 15)

ball_speed_x = 0
ball_speed_y = 0
p1_vel_y = 0
p2_vel_y = 0

p1_swing_timer = 0
p2_swing_timer = 0

score_p1 = 0
score_p2 = 0
font = pygame.font.Font(None, 50)
big_font = pygame.font.Font(None, 80)
guide_font = pygame.font.Font(None, 30)

game_over = False
winner_text = ""
last_hitter = 0 
hit_cooldown = 0
smash_effect = 0 

# 서브 관련
is_serving = True      
current_server = 1     # 1: Player 1, -1: Player 2

def prepare_serve(server):
    global is_serving, current_server, ball_speed_x, ball_speed_y, last_hitter, hit_cooldown
    is_serving = True
    current_server = server
    ball_speed_x = 0
    ball_speed_y = 0
    last_hitter = 0
    hit_cooldown = 0

def check_game_over():
    global game_over, winner_text
    if score_p1 >= 11:
        game_over = True
        winner_text = "PLAYER 1 WINS!"
    elif score_p2 >= 11:
        game_over = True
        winner_text = "PLAYER 2 WINS!"

# 타격 물리 계산
def calculate_hit_velocity(hitter_rect, ball_rect, is_jump, direction):
    relative_y = ball_rect.centery - hitter_rect.centery
    speed_x = 0
    speed_y = 0
    is_smash = False

    # 1. 스매시: 점프 + 머리 위 타격
    if is_jump and relative_y < -10:
        speed_x = 26 * direction
        speed_y = 5   
        is_smash = True
    
    # 2. 클리어
    else:
        speed_x = 14 * direction 
        speed_y = -14
        
    return speed_x, speed_y, is_smash

prepare_serve(1)

# --- 메인 루프 ---
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        
        if event.type == pygame.KEYDOWN:
            if game_over:
                if event.key == pygame.K_r:
                    score_p1 = 0
                    score_p2 = 0
                    game_over = False
                    prepare_serve(1) 
            else:
                # --- Player 1 조작 (WASD) ---
                if event.key == pygame.K_w and player1.bottom >= HEIGHT:
                    p1_vel_y = JUMP_STRENGTH
                
                if event.key == pygame.K_s:
                    if is_serving and current_server == 1:
                        ball_speed_x = 14 * 1
                        ball_speed_y = -14
                        is_serving = False
                        last_hitter = 1
                        hit_cooldown = 15
                    elif not is_serving and p1_swing_timer == 0:
                        p1_swing_timer = SWING_DURATION

                # --- Player 2 조작 (방향키) ---
                if event.key == pygame.K_UP and player2.bottom >= HEIGHT:
                    p2_vel_y = JUMP_STRENGTH
                
                if event.key == pygame.K_DOWN:
                    if is_serving and current_server == -1:
                        ball_speed_x = 14 * -1
                        ball_speed_y = -14
                        is_serving = False
                        last_hitter = 2
                        hit_cooldown = 15
                    elif not is_serving and p2_swing_timer == 0:
                        p2_swing_timer = SWING_DURATION

    if not game_over:
        keys = pygame.key.get_pressed()
        
        # P1 이동
        if keys[pygame.K_a] and player1.left > 0:
            player1.x -= MOVE_SPEED
        if keys[pygame.K_d] and player1.right < WIDTH // 2 - 5:
            player1.x += MOVE_SPEED

        # P2 이동
        if keys[pygame.K_LEFT] and player2.left > WIDTH // 2 + 5:
            player2.x -= MOVE_SPEED
        if keys[pygame.K_RIGHT] and player2.right < WIDTH:
            player2.x += MOVE_SPEED

        # --- 물리 연산 ---
        p1_vel_y += GRAVITY
        player1.y += p1_vel_y
        if player1.bottom > HEIGHT: player1.bottom = HEIGHT; p1_vel_y = 0

        p2_vel_y += GRAVITY
        player2.y += p2_vel_y
        if player2.bottom > HEIGHT: player2.bottom = HEIGHT; p2_vel_y = 0

        if is_serving:
            if current_server == 1:
                ball.centerx = player1.centerx + 20
                ball.centery = player1.centery + 10
            else:
                ball.centerx = player2.centerx - 20
                ball.centery = player2.centery + 10
        else:
            # 공기 저항 및 중력
            ball_speed_x *= AIR_RESISTANCE 
            ball_speed_y *= AIR_RESISTANCE 
            ball_speed_y += GRAVITY
            
            ball.x += ball_speed_x
            ball.y += ball_speed_y

            # [핵심] 벽 튕기기 로직 (Wall Bounce)
            if ball.left < 0: 
                ball.left = 0
                ball_speed_x *= -0.7 # 벽에 맞으면 속도가 70%로 줄며 튕김
            if ball.right > WIDTH: 
                ball.right = WIDTH
                ball_speed_x *= -0.7
            if ball.top < 0: 
                ball.top = 0
                ball_speed_y *= -0.7

        if hit_cooldown > 0: hit_cooldown -= 1
        if smash_effect > 0: smash_effect -= 1

        # --- 충돌 처리 ---
        if hit_cooldown == 0 and not is_serving:
            # Player 1 Hit
            p1_hitbox = None
            if p1_swing_timer > 0:
                p1_swing_timer -= 1
                p1_hitbox = pygame.Rect(player1.centerx, player1.y - 20, 50, 60)
                if ball.colliderect(p1_hitbox):
                    if last_hitter == 1:
                        score_p2 += 1; prepare_serve(-1); check_game_over()
                    else:
                        is_jump = (player1.bottom < HEIGHT)
                        vx, vy, smash = calculate_hit_velocity(player1, ball, is_jump, 1)
                        ball_speed_x = vx; ball_speed_y = vy
                        last_hitter = 1; hit_cooldown = 15
                        if smash: smash_effect = 20

            # Player 2 Hit
            p2_hitbox = None
            if p2_swing_timer > 0:
                p2_swing_timer -= 1
                p2_hitbox = pygame.Rect(player2.centerx - 50, player2.y - 20, 50, 60)
                if ball.colliderect(p2_hitbox):
                    if last_hitter == 2:
                        score_p1 += 1; prepare_serve(1); check_game_over()
                    else:
                        is_jump = (player2.bottom < HEIGHT)
                        vx, vy, smash = calculate_hit_velocity(player2, ball, is_jump, -1)
                        ball_speed_x = vx; ball_speed_y = vy
                        last_hitter = 2; hit_cooldown = 15
        else:
             if p1_swing_timer > 0: p1_swing_timer -= 1
             if p2_swing_timer > 0: p2_swing_timer -= 1

        if ball.colliderect(net):
            ball_speed_x *= -1; ball.x += ball_speed_x

        # 바닥 충돌 (점수)
        if ball.bottom > HEIGHT:
            if ball.x < WIDTH // 2: 
                score_p2 += 1; prepare_serve(-1) 
            else: 
                score_p1 += 1; prepare_serve(1) 
            check_game_over()

    # --- 그리기 ---
    SCREEN.fill(BLACK)
    pygame.draw.rect(SCREEN, NET_COLOR, net)
    
    # Player 1
    pygame.draw.rect(SCREEN, BLUE, player1)
    if p1_swing_timer > 0: 
        start = (player1.centerx, player1.centery)
        end = (player1.centerx + 40, player1.y - 10)
        pygame.draw.line(SCREEN, YELLOW, start, end, 8) 
        pygame.draw.circle(SCREEN, YELLOW, end, 20, 3) 

    # Player 2
    pygame.draw.rect(SCREEN, RED, player2)
    if p2_swing_timer > 0:
        start = (player2.centerx, player2.centery)
        end = (player2.centerx - 40, player2.y - 10)
        pygame.draw.line(SCREEN, YELLOW, start, end, 8)
        pygame.draw.circle(SCREEN, YELLOW, end, 20, 3)

    ball_color = (255, 100, 100) if smash_effect > 0 else WHITE
    pygame.draw.circle(SCREEN, ball_color, ball.center, 8)

    score_text = font.render(f"P1 {score_p1} : {score_p2} P2", True, WHITE)
    SCREEN.blit(score_text, (WIDTH//2 - score_text.get_width()//2, 20))

    # 서브 안내
    if is_serving and not game_over:
        if current_server == 1:
            msg = guide_font.render("P1 Serve: Press 'S'", True, YELLOW)
            SCREEN.blit(msg, (player1.centerx - 60, player1.top - 40))
        else:
            msg = guide_font.render("P2 Serve: Press 'Down'", True, YELLOW)
            SCREEN.blit(msg, (player2.centerx - 60, player2.top - 40))

    if smash_effect > 0:
        smash_msg = big_font.render("SMASH!!", True, YELLOW)
        SCREEN.blit(smash_msg, (WIDTH//2 - smash_msg.get_width()//2, 150))

    if game_over:
        msg = big_font.render(winner_text, True, GREEN if score_p1 > score_p2 else RED)
        SCREEN.blit(msg, (WIDTH//2 - msg.get_width()//2, HEIGHT//2 - 50))
        info = font.render("Press 'R' to Restart", True, WHITE)
        SCREEN.blit(info, (WIDTH//2 - info.get_width()//2, HEIGHT//2 + 50))

    pygame.display.flip()
    clock.tick(FPS)
