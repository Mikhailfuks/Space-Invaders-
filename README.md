
using namespace sf;

// Constants
const int windowWidth = 800;
const int windowHeight = 600;
const int invaderWidth = 32;
const int invaderHeight = 32;
const int playerWidth = 48;
const int playerHeight = 16;
const int bulletWidth = 4;
const int bulletHeight = 16;
const int invaderRows = 5;
const int invadersPerRow = 11;
const int maxBullets = 10;

// Colors
const Color backgroundColor = Color(0, 0, 0);
const Color playerColor = Color(255, 255, 255);
const Color bulletColor = Color(255, 0, 0);
const Color invaderColor = Color(0, 255, 0);

// Sounds
sf::SoundBuffer buffer;
sf::Sound sound;

// Structures
struct Invader {
    RectangleShape shape;
    bool alive;
    Vector2f velocity;
};

struct Bullet {
    RectangleShape shape;
    bool isPlayerBullet;
    Vector2f velocity;
};

// Global variables
vector<Invader> invaders;
vector<Bullet> bullets;
RectangleShape player;
float playerX = windowWidth / 2 - playerWidth / 2;
float playerY = windowHeight - playerHeight - 20;
bool gameRunning = true;

// Function to create an invader
Invader createInvader(int row, int col) {
    Invader invader;
    invader.shape.setSize(Vector2f(invaderWidth, invaderHeight));
    invader.shape.setFillColor(invaderColor);
    invader.shape.setPosition(col * invaderWidth + 50, row * invaderHeight + 50);
    invader.alive = true;
    invader.velocity = Vector2f(100.0f, 0.0f);
    return invader;
}

// Function to create a bullet
Bullet createBullet(float x, float y, bool isPlayerBullet) {
    Bullet bullet;
    bullet.shape.setSize(Vector2f(bulletWidth, bulletHeight));
    bullet.shape.setFillColor(bulletColor);
    bullet.shape.setPosition(x, y);
    bullet.isPlayerBullet = isPlayerBullet;
    bullet.velocity = isPlayerBullet ? Vector2f(0.0f, -500.0f) : Vector2f(0.0f, 500.0f);
    return bullet;
}

// Function to handle player movement
void handlePlayerMovement(float deltaTime) {
    if (Keyboard::isKeyPressed(Keyboard::Left) && playerX > 0) {
        playerX -= 300.0f * deltaTime;
    }
    if (Keyboard::isKeyPressed(Keyboard::Right) && playerX < windowWidth - playerWidth) {
        playerX += 300.0f * deltaTime;
    }
    player.setPosition(playerX, playerY);
}

// Function to handle bullet movement and collisions
void handleBulletMovement(float deltaTime) {
    for (int i = 0; i < bullets.size(); i++) {
        Bullet& bullet = bullets[i];
