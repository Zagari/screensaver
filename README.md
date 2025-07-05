# Zagari Screensaver - Raspberry Pi 2

Um protetor de tela personalizado otimizado para Raspberry Pi 2 com Debian/Raspbian, apresentando efeitos visuais animados com o texto qualquer como "ZAGARI".

## 🎯 Características

- **Otimizado para RPi 2**: Baixo consumo de CPU e memória
- **Múltiplos efeitos**: 5 efeitos visuais diferentes
- **Framebuffer direto**: Funciona sem X11 quando possível
- **Auto-rotação**: Muda efeitos automaticamente
- **Controle manual**: Troca efeitos com tecla SPACE
- **Inicialização automática**: Serviço systemd configurável

## 🎨 Efeitos Disponíveis

1. **Bouncing**: Texto saltitante que muda de cor ao colidir
2. **Fade**: Efeito de fade in/out com mudança gradual de cores
3. **Orbital**: Movimento circular com rotação de cores
4. **Matrix**: Múltiplas instâncias caindo como "Matrix"
5. **Wave**: Movimento senoidal horizontal

## 🔧 Requisitos

- Raspberry Pi 2 ou superior
- Debian/Raspbian OS
- Python 3.6+
- pygame
- Acesso ao framebuffer (`/dev/fb0`)

## 📦 Instalação Rápida

```bash
# 1. Baixar os arquivos
git clone https://github.com/seu-usuario/screensaver.git
cd screensaver

# 2. Tornar executável e instalar
chmod +x install.sh
./install.sh

# 3. Reiniciar o sistema
sudo reboot
```

## 🚀 Instalação Manual

### Passo 1: Dependências

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Python e pygame
sudo apt install -y python3 python3-pip python3-pygame

# Instalar dependências SDL
sudo apt install -y libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev
```

### Passo 2: Configurar Framebuffer

```bash
# Adicionar usuário ao grupo video
sudo usermod -a -G video $USER

# Configurar permissões do framebuffer
sudo chmod 666 /dev/fb0

# Criar regra udev
echo 'SUBSYSTEM=="graphics", KERNEL=="fb*", MODE="0666"' | sudo tee /etc/udev/rules.d/99-framebuffer.rules
```

### Passo 3: Otimizações do Sistema

```bash
# Configurar GPU memory split
echo "gpu_mem=16" | sudo tee -a /boot/config.txt

# Reduzir swappiness
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# Desabilitar serviços desnecessários
sudo systemctl disable bluetooth.service
sudo systemctl disable cups.service
sudo systemctl disable avahi-daemon.service
```

### Passo 4: Instalar Screensaver

```bash
# Criar diretório
mkdir -p ~/screensaver
cd ~/screensaver

# Copiar arquivos (screensaver.py e config.py)
# ... copiar os arquivos ...

# Tornar executável
chmod +x screensaver.py
```

## 🎮 Uso

### Execução Manual

```bash
cd ~/screensaver
python3 screensaver.py
```

### Controles

- **ESC ou Q**: Sair do screensaver
- **SPACE**: Trocar efeito manualmente
- **Qualquer tecla**: Detecta atividade (pode ser configurado)

### Execução Automática

```bash
# Ativar serviço systemd
sudo systemctl enable screensaver
sudo systemctl start screensaver

# Verificar status
sudo systemctl status screensaver

# Ver logs
journalctl -u screensaver -f
```

## ⚙️ Configuração

Edite o arquivo `config.py` para personalizar:

```python
# Configurações de display
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
FPS = 20  # Reduzido para economizar CPU

# Configurações de efeitos
EFFECT_DURATION = 15000  # 15 segundos por efeito
FONT_SIZE = 72

# Configurações de energia
IDLE_TIMEOUT = 600  # 10 minutos até ativar
SLEEP_TIMEOUT = 1800  # 30 minutos até desligar display
```

## 🔧 Resolução de Problemas

### Erro: "No module named pygame"

```bash
# Reinstalar pygame
sudo apt install -y python3-pygame
# ou
pip3 install pygame
```

### Erro: "Permission denied /dev/fb0"

```bash
# Verificar se usuário está no grupo video
groups $USER

# Adicionar ao grupo se necessário
sudo usermod -a -G video $USER
sudo reboot
```

### Erro: "Could not initialize SDL"

```bash
# Verificar se framebuffer está disponível
ls -la /dev/fb0

# Configurar variáveis de ambiente
export SDL_FBDEV=/dev/fb0
export SDL_VIDEODRIVER=fbcon
export SDL_NOMOUSE=1
```

### Performance Lenta

```bash
# Reduzir FPS no config.py
FPS = 15  # ou menor

# Verificar temperatura da CPU
vcgencmd measure_temp

# Verificar uso de memória
free -h
```

## 🛠️ Desenvolvimento

### Estrutura do Projeto

```
screensaver/
├── screensaver.py  # Script principal
├── config.py             # Configurações
├── install.sh            # Instalador
├── start_screensaver.sh  # Script de inicialização
├── logs/                 # Logs do sistema
│   └── screensaver.log
└── README.md            # Este arquivo
```

### Adicionando Novos Efeitos

Para criar um novo efeito, adicione no arquivo `screensaver.py`:

```python
# 1. Adicionar novo tipo de efeito
class EffectType(Enum):
    # ... efeitos existentes ...
    SPIRAL = 6  # Novo efeito

# 2. Criar método de inicialização
def init_spiral_effect(self):
    """Efeito espiral"""
    self.text_objects.append(TextObject(
        x=SCREEN_WIDTH // 2,
        y=SCREEN_HEIGHT // 2,
        dx=0, dy=0,
        alpha=255,
        color=COLORS['RED'],
        angle=0,
        scale=0.5
    ))

# 3. Criar método de atualização
def update_spiral_effect(self, obj: TextObject):
    """Atualiza efeito espiral"""
    time_factor = (pygame.time.get_ticks() - self.effect_start_time) / 1000.0
    
    # Espiral crescente
    radius = 50 + time_factor * 20
    obj.angle += 0.1
    
    center_x = SCREEN_WIDTH // 2
    center_y = SCREEN_HEIGHT // 2
    
    obj.x = center_x + radius * math.cos(obj.angle) - self.text_rect.width // 2
    obj.y = center_y + radius * math.sin(obj.angle) - self.text_rect.height // 2
    
    # Escala baseada no tempo
    obj.scale = 0.5 + 0.5 * math.sin(time_factor)
```

### Personalização de Cores

```python
# Adicionar cores personalizadas
CUSTOM_COLORS = {
    'ZAGARI_BLUE': (0, 150, 255),
    'ZAGARI_GREEN': (50, 255, 150),
    'ZAGARI_PURPLE': (200, 50, 255),
    'ZAGARI_ORANGE': (255, 150, 50)
}

# Usar no código
obj.color = CUSTOM_COLORS['ZAGARI_BLUE']
```

## 🎛️ Configurações Avançadas

### Otimização para Diferentes Resoluções

```python
# Para display 1024x768
SCREEN_WIDTH = 1024
SCREEN_HEIGHT = 768
FONT_SIZE = 90

# Para display 640x480 (mais leve)
SCREEN_WIDTH = 640
SCREEN_HEIGHT = 480
FONT_SIZE = 48
FPS = 15
```

### Configuração de Energia

```python
# Configurar desligamento automático do display
import subprocess

def turn_off_display():
    subprocess.run(['vcgencmd', 'display_power', '0'])

def turn_on_display():
    subprocess.run(['vcgencmd', 'display_power', '1'])
```

### Detecção de Atividade

```python
import select
import sys

def check_keyboard_activity():
    """Verifica se há atividade no teclado"""
    if select.select([sys.stdin], [], [], 0) == ([sys.stdin], [], []):
        return True
    return False
```

## 🚀 Scripts Úteis

### Script de Monitoramento

```bash
#!/bin/bash
# monitor_screensaver.sh
# Monitora performance do screensaver

while true; do
    echo "=== $(date) ==="
    echo "CPU Temperature: $(vcgencmd measure_temp)"
    echo "Memory Usage:"
    free -h
    echo "GPU Memory Split:"
    vcgencmd get_mem arm
    vcgencmd get_mem gpu
    echo "Screensaver Process:"
    ps aux | grep screensaver | grep -v grep
    echo "------------------------"
    sleep 30
done
```

### Script de Backup

```bash
#!/bin/bash
# backup_config.sh
# Backup das configurações

BACKUP_DIR="/home/pi/screensaver-backup-$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Backup dos arquivos
cp ~/screensaver/*.py "$BACKUP_DIR/"
cp ~/screensaver/config.py "$BACKUP_DIR/"
cp /etc/systemd/system/screensaver.service "$BACKUP_DIR/"

# Backup das configurações do sistema
cp /boot/config.txt "$BACKUP_DIR/"
cp /etc/sysctl.conf "$BACKUP_DIR/"

echo "Backup salvo em: $BACKUP_DIR"
```

## 🔍 Logs e Debugging

### Visualizar Logs

```bash
# Logs do sistema
journalctl -u screensaver -f

# Logs do arquivo
tail -f ~/screensaver/logs/screensaver.log

# Logs de erro
journalctl -u screensaver --priority=err
```

### Debug Mode

Adicione no início do `screensaver.py`:

```python
import logging

# Configurar logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('debug.log'),
        logging.StreamHandler()
    ]
)

# Usar no código
logging.debug("Iniciando efeito bouncing")
logging.info(f"FPS atual: {self.clock.get_fps()}")
```

## 🎨 Customizações Visuais

### Fontes Personalizadas

```python
# Usar fonte personalizada
try:
    self.font = pygame.font.Font("/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf", self.font_size)
except:
    self.font = pygame.font.Font(None, self.font_size)  # Fallback
```

### Efeitos de Partículas

```python
class Particle:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.dx = random.uniform(-2, 2)
        self.dy = random.uniform(-2, 2)
        self.life = 255
        self.color = random.choice(list(COLORS.values()))
    
    def update(self):
        self.x += self.dx
        self.y += self.dy
        self.life -= 2
        
    def draw(self, screen):
        if self.life > 0:
            pygame.draw.circle(screen, self.color, (int(self.x), int(self.y)), 3)
```

## 📱 Controle Remoto

### Via SSH

```bash
# Conectar via SSH
ssh pi@192.168.1.100

# Controlar screensaver
sudo systemctl start screensaver
sudo systemctl stop screensaver
sudo systemctl restart screensaver
```

### Via API Web (opcional)

```python
from flask import Flask, render_template
app = Flask(__name__)

@app.route('/control/<action>')
def control_screensaver(action):
    if action == 'start':
        # Iniciar screensaver
        pass
    elif action == 'stop':
        # Parar screensaver
        pass
    return f"Ação: {action}"
```

## 🔐 Segurança

### Executar como Serviço Não-Root

```bash
# Criar usuário específico
sudo useradd -r -s /bin/false screen-user
sudo usermod -a -G video screen-user

# Configurar permissões
sudo chown -R screen-user:screen-user /home/pi/screensaver
```

### Limitações de Recursos

```bash
# Configurar limits no systemd
[Service]
MemoryMax=100M
CPUQuota=50%
```

## 🆘 Suporte e Contribuições

### Reportar Bugs

1. Verifique se o problema persiste após reiniciar
2. Colete logs: `journalctl -u screensaver --since "1 hour ago"`
3. Inclua informações do sistema: `uname -a`, `free -h`, `vcgencmd measure_temp`

### Contribuir

1. Fork o projeto
2. Crie uma branch para sua feature
3. Teste em Raspberry Pi 2
4. Envie um pull request

## 📚 Referências

- [pygame Documentation](https://www.pygame.org/docs/)
- [Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/)
- [SDL2 Documentation](https://wiki.libsdl.org/SDL2)
- [systemd Service Documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

---

**Zagari Screensaver** - Transforme seu Raspberry Pi em um display artístico! 🎨