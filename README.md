# game_shooter
from pygame import *
from random import randint

mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')

font.init()
font1 = font.Font(None, 36)
win = font1.render('YOU WIN!', True, (255, 255, 255))
lose = font1.render('YOU LOSE!', True, (180, 0, 0))
font2 = font.Font(None, 36)

img_bullet = "bullet.png"

score = 0
lost = 0
max_lost = 3

win_width = 700
win_height = 500
window = display.set_mode((win_width, win_height))
display.set_caption("Шутер")
background = transform.scale(image.load('galaxy.jpg'), (win_width, win_height))

class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        super().__init__()
        self.image = transform.scale(image.load(player_image), (65, 65))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
    def fire(self):
        bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top, 15, 20, -15)
        bullets.add(bullet)

class Monsters(GameSprite):
    def update(self):
        self.rect.y += self.speed 
        global lost 
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost = lost + 1


class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed 
        if self.rect.y < 0:
            self.kill()

player = Player('rocket.png', 5, win_height - 80, 5, 5, 10)

monsters = sprite.Group()
for i in range(1, 6):
    monster = Monsters('ufo.png', randint(80, win_width - 80), -40, 80, 50, randint(1, 2))
    monsters.add(monster)

bullets = sprite.Group()

clock = time.Clock()
FPS = 60

finish = False

run = True
while run:
    for e in event.get():
        if e.type == QUIT:
            run = False
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:
                fire_sound.play()
                player.fire()
    if finish != True:
        window.blit(background, (0, 0))
        sprite.spritecollide(player, monsters, False)
        collides = sprite.groupcollide(monsters, bullets, True, True)

        for c in collides:
            score+=1
            monster = Monsters('ufo.png', randint(80, win_width - 80), -40, 80, 50, randint(1, 2))
            monsters.add(monster)

        text_lose = font1.render("Пропущено: " + str(lost), 1, (200, 200, 200))
        window.blit(text_lose, (10, 30))
        text_defeat = font1.render("Убито: " + str(score) , 1, (200, 200, 200))
        window.blit(text_defeat, (10, 50))
        if lost >= 3:
            finish = True
            window.blit(lose, (250, 250))

        if score >= 10:
            finish = True
            window.blit(win, (250, 250))     

        player.update()
        monsters.update()
        bullets.update()

        player.reset()
        monsters.draw(window)
        bullets.draw(window)

    display.update()
    clock.tick(FPS)
