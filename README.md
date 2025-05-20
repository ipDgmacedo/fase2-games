import pygame
import sys
import random

# Inicialização
pygame.init()
largura, altura = 1024, 768
tela = pygame.display.set_mode((largura, altura))
pygame.display.set_caption("Pac-Man com Mapa Cheio e Vários NPCs")
clock = pygame.time.Clock()

#imagens:
img_barracas = pygame.image.load('31bc63e8-877b-4f59-b8ea-b76f4c18e720.jpg')
img_barracas = pygame.transform.scale(img_barracas, (100, 100))




# Cores
PRETO = (0, 0, 0)
AZUL = (0, 0, 255)
AMARELO = (255, 255, 0)
BRANCO = (255, 255, 255)
VERMELHO = (255, 0, 0)
ROSA = (255, 105, 180)
CIANO = (0, 255, 255)
LARANJA = (255, 165, 0)
VERDE = (0, 255, 0)

# Mapa
mapa_base = [
    "###################################################",
    "#                                                 #",
    "# ####  ####   ####### ######   #                 #",
    "# #  #  #  #   #     # #    #   #  #########      #",
    "# #  #  #  #   ####### ###  #   #  #       #   #  #",
    "# ####  ####             ####      #       #   #  #",
    "#                                 ##       #   #  #",
    "# #######    #####   #####  ##    #        #   #  #",
    "# #     #    #   #   #   #  ##    ##########   #  #",
    "# #######    #   #   #   #  ##                    #",
    "#            #####   #####        ###########     #",
    "########                                          #",
    "#         ####### ####### #########          ###  #",
    "#   #     #     # #     # #       #   #####  # #  #",
    "#   #     ####### ####### #########          # #  #",
    "#   #                                        # #  #",
    "#   ##             ####### ###### ########   ###  #",
    "#      ########                                   #",
    "# ######      ####  #####     ##############      #",
    "# #              #  #   #     #           ##      #",
    "# ################  #####     #############    XXX#",
    "#                                              XXX#",
    "###################################################"
]

# Normalizar mapa
colunas = max(len(linha) for linha in mapa_base)
mapa_base = [linha.ljust(colunas, "#") for linha in mapa_base]
linhas = len(mapa_base)

# Blocos
tamanho_bloco_x = largura // colunas
tamanho_bloco_y = altura // linhas

# Pac-Man
pacman_x, pacman_y = 1, 1
direcao_x, direcao_y = 0, 0

# NPCs
npcs = [
    {"x": colunas - 2, "y": linhas - 2, "cor": VERMELHO},
    {"x": 1, "y": linhas - 2, "cor": ROSA},
    {"x": colunas - 2, "y": 1, "cor": CIANO},
    {"x": colunas // 2, "y": linhas // 2, "cor": LARANJA},
]
npc_timer = 0

def desenhar_mapa():
    for y, linha in enumerate(mapa_base):
        for x, caractere in enumerate(linha):
            rect = pygame.Rect(x * tamanho_bloco_x, y * tamanho_bloco_y, tamanho_bloco_x, tamanho_bloco_y)
            if caractere == "#":
                pygame.draw.rect(tela, AZUL, rect)
            elif caractere == ".":
                pygame.draw.circle(tela, BRANCO, rect.center, min(tamanho_bloco_x, tamanho_bloco_y) // 6)
            elif caractere == "X":
                pygame.draw.rect(tela, VERDE, rect)

def mover_pacman():
    global pacman_x, pacman_y
    novo_x = pacman_x + direcao_x
    novo_y = pacman_y + direcao_y
    if 0 <= novo_x < colunas and 0 <= novo_y < linhas:
        if mapa_base[novo_y][novo_x] != "#":
            pacman_x = novo_x
            pacman_y = novo_y

def atualizar_mapa():
    linha = list(mapa_base[pacman_y])
    if linha[pacman_x] == ".":
        linha[pacman_x] = " "
        mapa_base[pacman_y] = "".join(linha)

def mover_npcs():
    for npc in npcs:
        dx = pacman_x - npc["x"]
        dy = pacman_y - npc["y"]

        direcoes = []
        if abs(dx) > abs(dy):
            if dx != 0:
                direcoes.append((1 if dx > 0 else -1, 0))
            if dy != 0:
                direcoes.append((0, 1 if dy > 0 else -1))
        else:
            if dy != 0:
                direcoes.append((0, 1 if dy > 0 else -1))
            if dx != 0:
                direcoes.append((1 if dx > 0 else -1, 0))

        random.shuffle(direcoes)

        for dx, dy in direcoes:
            nx = npc["x"] + dx
            ny = npc["y"] + dy
            if 0 <= ny < len(mapa_base) and 0 <= nx < len(mapa_base[ny]):
                if mapa_base[ny][nx] != "#":
                    npc["x"] = nx
                    npc["y"] = ny
                    break

def verificar_colisao():
    for npc in npcs:
        if pacman_x == npc["x"] and pacman_y == npc["y"]:
            return True
    return False

def verificar_vitoria():
    return mapa_base[pacman_y][pacman_x] == "X"

# Loop principal
rodando = True
while rodando:
    clock.tick(10)
    tela.fill(PRETO)  # Ainda pode usar para limpar a tela, mas a imagem cobrirá

    # Desenha a imagem das barracas primeiro, cobrindo toda a tela
    tela.blit(img_barracas, (0, 0))

    # Em seguida, desenha o mapa por cima
    desenhar_mapa()

    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            rodando = False
        elif evento.type == pygame.KEYDOWN:
            if evento.key == pygame.K_LEFT:
                direcao_x, direcao_y = -1, 0
            elif evento.key == pygame.K_RIGHT:
                direcao_x, direcao_y = 1, 0
            elif evento.key == pygame.K_UP:
                direcao_x, direcao_y = 0, -1
            elif evento.key == pygame.K_DOWN:
                direcao_x, direcao_y = 0, 1

    mover_pacman()
    atualizar_mapa()

    npc_timer += 1
    if npc_timer > 1:
        mover_npcs()
        npc_timer = 0

    if verificar_colisao():
        print("Você foi pego!")
        rodando = False

    if verificar_vitoria():
        print("Você venceu o jogo!")
        rodando = False

    # Desenhar Pac-Man
    pygame.draw.circle(tela, AMARELO,
                       (pacman_x * tamanho_bloco_x + tamanho_bloco_x // 2,
                        pacman_y * tamanho_bloco_y + tamanho_bloco_y // 2),
                       min(tamanho_bloco_x, tamanho_bloco_y) // 2 - 2)

    # Desenhar todos os NPCs
    for npc in npcs:
        pygame.draw.circle(tela, npc["cor"],
                           (npc["x"] * tamanho_bloco_x + tamanho_bloco_x // 2,
                            npc["y"] * tamanho_bloco_y + tamanho_bloco_y // 2),
                           min(tamanho_bloco_x, tamanho_bloco_y) // 2 - 2)

    pygame.display.flip()

pygame.quit()
sys.exit()
