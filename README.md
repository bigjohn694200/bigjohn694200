import pygame
import random
import time
"""


"""
res = 528, 940
FPS = 60

pygame.init()
screen = pygame.display.set_mode(res)
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()

bg = pygame.image.load('img/bg.png').convert()
bg_img = pygame.transform.scale(bg, (528, 940))

main_font = pygame.font.Font("font/mainFont.ttf", 90)
title_name = main_font.render(
    "Flappy Bird\nPress Enter \n\tto Play \t \n\nPress Space\n To Jump", True, pygame.Color("white"))

score = 0
high_score = 0

ground_image = pygame.image.load(
    "img/fb_ground.png").convert_alpha()  # Charger l'image du sol
ground_x = 0  # Position x du sol
ground_y = 450  # Position y du sol
ground_velocity = 0  # Vitesse de déplacement du sol

player_surf = pygame.image.load("img/bird.png").convert_alpha()
player_surf = pygame.transform.scale(player_surf, (79, 55))
player_rect = player_surf.get_rect(topleft=(220, 400))

# Charger l'image du tuyau supérieur
pipe_upper_image = pygame.image.load("img/pipe_upper.png")
pipe_upper_image = pygame.transform.scale(pipe_upper_image, (256, 360))
pipe_upper_rect = pipe_upper_image.get_rect()
new_pipe = pipe_upper_image.get_rect()
# Charger l'image du tuyau inférieur
pipe_lower_image = pygame.image.load("img/pipe_lower.png")
pipe_lower_image = pygame.transform.scale(pipe_lower_image, (256, 360))
pipe_lower_rect = pipe_lower_image.get_rect()

pipe_upper_mask = pygame.mask.from_surface(pipe_upper_image)
pipe_lower_mask = pygame.mask.from_surface(pipe_lower_image)

pipe_x = 528  # Position x du tuyau (commencer juste à l'extérieur de l'écran)
pipe_x2 = 925
pipe_gap = 625  # Écart vertical entre les tuyaux supérieur et inférieur
# Largeur du tuyau (utilisez l'image du tuyau supérieur pour obtenir la largeur)
pipe_width = pipe_upper_image.get_width()
pipe_y_range = (375, 600)
pipe_y = 500
pipe_y2 = 400

tilt_angle_up = 25  # Angle d'inclinaison vers le haut (en degrés)
tilt_angle_down = -25  # Angle d'inclinaison vers le bas (en degrés)
tilt_frames = 10
# Angle d'inclinaison actuel du joueur (initialement à zéro)
player_tilt_angle = 0
frame_index = 0
pygame.mixer.init()
flap_sound = pygame.mixer.Sound("sound/flap.mp3")
hit_sound = pygame.mixer.Sound("sound/hit.mp3")
score_sound = pygame.mixer.Sound("sound/point.mp3")
die_sound = pygame.mixer.Sound("sound/die.mp3")

player_gravity = 0
passed_pipe = False
show_title = True
score_increment_interval = 1.65
score_increment_allowed = True
last_increment_time = time.time()

playing = False

running = True
while running:
    screen.blit(bg_img, (0, 0))
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and playing:
                flap_sound.play()
                player_gravity = -15
            if event.key == pygame.K_RETURN:
                playing = True
                show_title = False

    current_time = time.time()

    ground_x -= ground_velocity  # Mettre à jour la position x du sol
    if ground_x <= -520:  # Réinitialiser la position du sol lorsqu'il sort de l'écran
        ground_x = 0

    player_gravity += 0.75
    player_rect.y += player_gravity

    if high_score < score:
        high_score = score
    elif high_score >= score:
        high_score = high_score
    
    if player_rect.y >= 690 or player_rect.y <=-100 and playing:
        hit_sound.play()
        player_gravity = 0
        ground_velocity = 0
        pipe_x, pipe_x2 = 528, 925
        playing = False
        player_rect.y = 400
        show_title = True
    else:
        if playing:
            ground_velocity = 5
        else:
            ground_velocity = 0
            player_gravity = 0
    
    
    if player_gravity >= 4:
        target_tilt_angle = tilt_angle_down
    else:
        target_tilt_angle = tilt_angle_up

    if player_tilt_angle != target_tilt_angle:
        angle_difference = target_tilt_angle - player_tilt_angle
        intermediate_tilt_angle = player_tilt_angle + (angle_difference / tilt_frames)
        player_tilt_angle = intermediate_tilt_angle

    rotated_player_surf = pygame.transform.rotate(player_surf, player_tilt_angle)

    pipe_x -= ground_velocity  # Mettre à jour la position x du tuyau
    pipe_x2 -= ground_velocity

    if pipe_x + pipe_width < 0:  # Réinitialiser la position du tuyau lorsqu'il sort de l'écran
        pipe_x = 528
        pipe_y = random.randrange(pipe_y_range[0], pipe_y_range[1])
    if pipe_x2 + pipe_width < 0:  # Réinitialiser la position du tuyau lorsqu'il sort de l'écran
        pipe_x2 = 528
        pipe_y2 = random.randrange(pipe_y_range[0], pipe_y_range[1])

    player_mask = pygame.mask.from_surface(player_surf)

    if player_mask.overlap(pipe_upper_mask, (pipe_x - player_rect.x, pipe_y - player_rect.y)) or \
       player_mask.overlap(pipe_lower_mask, (pipe_x - player_rect.x, pipe_y - pipe_gap - player_rect.y)):
        # Collision détectée avec un tuyau
        # Gérer l'action à prendre lorsque le joueur entre en collision avec un tuyau
        hit_sound.play()
        die_sound.play
        show_title = True
        playing = False
        score = 0
        pipe_x = 528
        pipe_x2 = 925
        player_rect.y = 400
    if player_mask.overlap(pipe_upper_mask, (pipe_x2 - player_rect.x, pipe_y2 - player_rect.y)) or \
       player_mask.overlap(pipe_lower_mask, (pipe_x2 - player_rect.x, pipe_y2 - pipe_gap - player_rect.y)):
        # Collision détectée avec un tuyau
        # Gérer l'action à prendre lorsque le joueur entre en collision avec un tuyau
        hit_sound.play()
        show_title = True
        playing = False
        score = 0
        pipe_x = 528
        pipe_x2 = 925
        player_rect.y = 400

    if score_increment_allowed and pipe_x in range(105, 110) or score_increment_allowed and pipe_x2 in range(105, 110):
        score += 1
        score_sound.play()
        # Désactive l'incrémentation du score pendant un certain temps
        score_increment_allowed = False

    if not score_increment_allowed and time.time() - last_increment_time >= score_increment_interval:
        score_increment_allowed = True

    # Dessiner le tuyau supérieur
    screen.blit(pipe_upper_image, (pipe_x, pipe_y))
    # Dessiner le tuyau inférieur
    screen.blit(pipe_lower_image, (pipe_x, pipe_y - pipe_gap))

    screen.blit(pipe_upper_image, (pipe_x2, pipe_y2))
    screen.blit(pipe_lower_image, (pipe_x2, pipe_y2 - pipe_gap))
  
    screen.blit(ground_image, (ground_x, ground_y))
    #rotated_player_surf = pygame.transform.rotate(
        #player_surf, player_tilt_angle)
    rotated_player_rect = rotated_player_surf.get_rect(
        center=player_rect.center)
    screen.blit(rotated_player_surf, rotated_player_rect)
    # screen.blit(player_surf, player_rect)
    if show_title:
        screen.blit(title_name, (80, 125))
    screen.blit(main_font.render(str(score), True,
                pygame.Color("white")), (240, 50))
    screen.blit(main_font.render(f"Highscore : {high_score}", True,
                pygame.Color("red")), (25, 825))
    pygame.display.update()
    # pygame.display.flip()
    clock.tick(FPS)

"""

"""
