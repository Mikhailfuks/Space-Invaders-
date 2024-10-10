
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


        bullet.shape.move(bullet.velocity * deltaTime);
        if (bullet.shape.getPosition().y < 0 || bullet.shape.getPosition().y > windowHeight) {
            bullets.erase(bullets.begin() + i);
            i--;
        } else {
            // Check for collisions with invaders
            for (int j = 0; j < invaders.size(); j++) {
                Invader& invader = invaders[j];
                if (invader.alive && bullet.shape.getGlobalBounds().intersects(invader.shape.getGlobalBounds())) {
                    invader.alive = false;
                    bullets.erase(bullets.begin() + i);
                    i--;
                    j--;
                    // Play explosion sound
                    sound.play();
                    break;
                }
            }
        }
    }
}

// Function to handle invader movement and collisions
void handleInvaderMovement(float deltaTime) {
    for (int i = 0; i < invaders.size(); i++) {
        Invader& invader = invaders[i];
        if (invader.alive) {
            invader.shape.move(invader.velocity * deltaTime);
            if (invader.shape.getPosition().x < 0 || invader.shape.getPosition().x > windowWidth - invaderWidth) {
                invader.velocity.x *= -1;
                for (Invader& inv : invaders) {
                    inv.shape.move(0.0f, invaderHeight);
                }
            }
        }
    }
}

int main() {
    // Create the window
    RenderWindow window(VideoMode(windowWidth, windowHeight), "Space Invaders");
    window.setFramerateLimit(60);

    // Load the explosion sound
    if (!buffer.loadFromFile("explosion.wav")) {
        cout << "Error loading sound file." << endl;
        return 1;
    }
    sound.setBuffer(buffer);

    // Create the player
    player.setSize(Vector2f(playerWidth, playerHeight));
    player.setFillColor(playerColor);
    player.setPosition(playerX, playerY);

    // Create the invaders
    for (int row = 0; row < invaderRows; row++) {
        for (int col = 0; col < invadersPerRow; col++) {
            invaders.push_back(createInvader(row, col));
        }
    }

    // Game loop
    Clock clock;
    while (window.isOpen() && gameRunning) {
        // Handle events
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }
            if (event.type == Event::KeyPressed) {
                if (event.key.code == Keyboard::Space) {
                    // Fire a bullet
                    bullets.push_back(createBullet(playerX + playerWidth / 2 - bulletWidth / 2, playerY, true));
                    if (bullets.size() > maxBullets) {
                        bullets.erase(bullets.begin());
                    }
                }
            }
        }

        // Update game logic
        float deltaTime = clock.restart().asSeconds();
        handlePlayerMovement(deltaTime);
        handleBulletMovement(deltaTime);
        handleInvaderMovement(deltaTime);

        // Check for game over
        bool allInvadersDead = true;
        for (const Invader& invader : invaders) {
            if (invader.alive) {
                allInvadersDead = false;
                break;
            }
        }
        if (allInvadersDead) {
            displayMessage("You win!");
            gameRunning = false;
        }

        // Draw everything
        window.clear(backgroundColor);
        for (const Bullet& bullet : bullets) {
            window.draw(bullet.shape);
        }
        for (const Invader& invader : invaders) {
            if (invader.alive) {
                window.draw(invader.shape);
            }
        }
        window.draw(player);
        window.display();
    }

    return 0;
}
