# A Word-based Fighting Game (First Major Game)

A game where you fight enemies, drink potions and special potions  
that can have positive or negative effects. You can purchase weapons  
to strengthen yourself, and your goal is to defeat the fearsome Trip-Headed Wolf.

## Features
- Text-based combat system
- Potions with both good and bad effects
- Weapon purchasing system
- Final boss: The Trip-Headed Wolf

## How to Play
- Clone the repository or download the code
- Compile with a C++ compiler - I use Programiz - https://www.programiz.com/cpp-programming/online-compiler/
- Run and enjoy!

- Code Below 

#include <iostream>
#include <cstdlib>
#include <ctime>
#include <string>
#include <vector>
#include <iomanip>
#include <unistd.h>
using namespace std;

struct Sword {
    string name;
    int bonusDamage;
};

struct Player {
    string name;
    int health = 100;
    int gold = 0;
    int potions = 3;
    int mysteriousPotions = 1;
    int tempAttackBonus = 0;
    vector<Sword> swords;
};

struct Enemy {
    string name;
    int health;
    int attack;
};

Enemy getRandomEnemy() {
    string enemies[] = {"Goblin", "Skeleton", "Orc", "Dark Knight", "Demon"};
    int health = 30 + rand() % 40;
    int attack = 5 + rand() % 10;
    return { enemies[rand() % 5], health, attack };
}

Enemy getFinalBoss() {
    return { "The Great Trip-Headed Wolf", 250, 25 };
}

void printHealthBar(string name, int health, int maxHealth) {
    int barWidth = 30;
    float healthPercentage = float(health) / maxHealth;
    int progress = barWidth * healthPercentage;

    cout << name << " Health: [";
    for (int i = 0; i < barWidth; i++) {
        if (i < progress) cout << "#";
        else cout << ".";
    }
    cout << "] " << health << "/" << maxHealth << endl;
}

void showStats(Player& player) {
    cout << "\nPlayer: " << player.name
         << "\nHealth: " << player.health
         << "\nGold: " << player.gold
         << "\nPotions: " << player.potions
         << "\nMysterious Potions: " << player.mysteriousPotions
         << "\nSwords Owned: ";
    if (player.swords.empty()) cout << "None";
    else {
        for (auto& sword : player.swords)
            cout << sword.name << " (+" << sword.bonusDamage << " dmg), ";
    }
    cout << "\n\n";
}

int getTotalSwordDamage(Player& player) {
    int bonus = 0;
    for (auto& sword : player.swords)
        bonus += sword.bonusDamage;
    return bonus + player.tempAttackBonus;
}

void useMysteriousPotion(Player& player) {
    if (player.mysteriousPotions <= 0) {
        cout << "You have no mysterious potions left.\n";
        return;
    }

    player.mysteriousPotions--;
    int effect = rand() % 4;

    switch (effect) {
        case 0:
            player.health += 30;
            if (player.health > 100) player.health = 100;
            cout << "[MYSTERIOUS POTION] You feel invigorated! (+30 Health)\n";
            break;
        case 1:
            player.tempAttackBonus += 10;
            cout << "[POWER] You feel power surge through you! (+10 attack bonus for next hit!)\n";
            break;
        case 2:
            player.health -= 20;
            cout << "[PAIN] The potion burns! (-20 Health)\n";
            break;
        case 3:
            player.tempAttackBonus -= 5;
            cout << "[WEAKNESS] You feel weak... (-5 attack for next battle!)\n";
            break;
    }
}

void battle(Player& player, Enemy& enemy) {
    cout << "You encountered a " << enemy.name << " (HP: " << enemy.health << ")!\n";

    while (enemy.health > 0 && player.health > 0) {
        cout << "\n[1] Attack\n[2] Use Potion\n[3] Use Mysterious Potion\n[4] Run (-5 gold)\n> ";
        int choice;
        cin >> choice;

        if (choice == 1) {
            int swordBonus = getTotalSwordDamage(player);
            int playerDmg = 10 + rand() % 10 + swordBonus;
            int enemyDmg = enemy.attack;
            enemy.health -= playerDmg;
            player.health -= enemyDmg;

            cout << "You hit the " << enemy.name << " for " << playerDmg << " damage!\n";
            cout << "It hits you for " << enemyDmg << " damage!\n";
            player.tempAttackBonus = 0; // clear temporary bonus after attack
            sleep(2);
        } else if (choice == 2 && player.potions > 0) {
            player.health += 25;
            if (player.health > 100) player.health = 100;
            player.potions--;
            cout << "You drank a potion and healed 25 health!\n";
            sleep(2);
        } else if (choice == 3) {
            useMysteriousPotion(player);
        } else if (choice == 4) {
            if (player.gold >= 5) {
                player.gold -= 5;
                cout << "You ran away and lost 5 gold.\n";
            } else {
                cout << "Not enough gold to run!\n";
                continue;
            }
            return;
        } else {
            cout << "[!] Invalid choice or no potions left.\n";
        }

        printHealthBar(player.name, player.health, 100);
        printHealthBar(enemy.name, enemy.health, 250);
    }

    if (player.health <= 0) {
        cout << "You died in battle...\n";
        return;
    } else {
        int loot = 10 + rand() % 20;
        cout << "You defeated the " << enemy.name << "! You found " << loot << " gold.\n";
        player.gold += loot;
        if (rand() % 2 == 0) {
            player.potions++;
            cout << "You also found a potion!\n";
        }
        if (rand() % 3 == 0) {
            player.mysteriousPotions++;
            cout << "You found a mysterious potion!\n";
        }
    }
}

void swordShop(Player& player) {
    Sword shop[5] = {
        {"Fire Sword", 5},
        {"Ice Blade", 4},
        {"Thunder Fang", 6},
        {"Shadow Dagger", 3},
        {"Crystal Cleaver", 7}
    };

    if (player.gold < 20) {
        cout << "You need at least 20 gold to buy a sword!\n";
        return;
    }

    if (player.swords.size() >= 5) {
        cout << "You can't carry more than 5 swords!\n";
        return;
    }

    cout << "\nSword Shop - Each sword costs 20 gold\n";
    for (int i = 0; i < 5; i++) {
        cout << "[" << i+1 << "] " << shop[i].name << " (+" << shop[i].bonusDamage << " dmg)\n";
    }

    cout << "[0] Cancel\n> ";
    int choice;
    cin >> choice;

    if (choice >= 1 && choice <= 5) {
        player.gold -= 20;
        player.swords.push_back(shop[choice - 1]);
        cout << "You bought the " << shop[choice - 1].name << "!\n";
    } else {
        cout << "Purchase canceled.\n";
    }
}

void startStoryline() {
    cout << "\n** Storyline Begin **\n";
    cout << "You are a brave adventurer, seeking glory in a world filled with dangers and treasures.\n";
    cout << "Your mission is to defeat The Great Trip-Headed Wolf, a terrible creature terrorizing the land!\n";
    cout << "You begin your journey in a dark dungeon...\n";
    cout << "** Storyline End **\n\n";
}

int main() {
    srand(time(0));
    Player player;

    cout << "Enter your name, Brave Adventurer: ";
    getline(cin, player.name);

    cout << "\nWelcome, " << player.name << "! Your journey begins...\n";
    startStoryline();

    while (player.health > 0) {
        showStats(player);
        cout << "[1] Explore Dungeon\n[2] Rest (10 gold, restores 10 health)\n[3] Buy Sword\n[4] Quit\n> ";
        int action;
        cin >> action;

        if (action == 1) {
            if (rand() % 10 == 0) {
                Enemy finalBoss = getFinalBoss();
                cout << "\nYou have entered a strange room. A massive trip-headed wolf appears!\n";
                battle(player, finalBoss);

                if (player.health <= 0) {
                    cout << "You died. The Great Trip-Headed Wolf was too powerful.\n";
                    break;
                } else {
                    cout << "You have defeated The Great Trip-Headed Wolf! The world is safe again!\n";
                    cout << "** YOU WIN **\n";
                    break;
                }
            } else {
                Enemy enemy = getRandomEnemy();
                battle(player, enemy);
            }
        } else if (action == 2) {
            if (player.gold >= 10) {
                player.gold -= 10;
                player.health += 10;
                if (player.health > 100) player.health = 100;
                cout << "You rested and recovered 10 health (10 gold spent).\n";
            } else {
                cout << "Not enough gold to rest!\n";
            }
        } else if (action == 3) {
            swordShop(player);
        } else if (action == 4) {
            cout << "Farewell, " << player.name << "! You retire with " << player.gold << " gold.\n";
            break;
        } else {
            cout << "[!] Invalid choice.\n";
        }
    }

    if (player.health <= 0) {
        cout << "\nGame Over. You collected " << player.gold << " gold.\n";
    }

    return 0;
}

