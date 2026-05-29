# RPG-Game
from dataclasses import dataclass, field


@dataclass
class Player:
    name: str
    health: int = 10
    gold: int = 3
    has_ember_key: bool = False
    has_moon_water: bool = False
    reputation: int = 0
    inventory: list[str] = field(default_factory=list)


def print_slow(text: str) -> None:
    print(f"\n{text}")


def ask_choice(prompt: str, choices: dict[str, tuple[str, str]]) -> str:
    while True:
        print_slow(prompt)

        for key, (label, _) in choices.items():
            print(f"  {key}. {label}")

        answer = input("\nChoose an option: ").strip().lower()

        if answer in choices:
            return choices[answer][1]

        print("\nInvalid choice. Please try again.")


def show_status(player: Player) -> None:
    inventory = ", ".join(player.inventory) if player.inventory else "empty"

    print(
        f"\n--- {player.name}'s Status ---\n"
        f"Health: {player.health}\n"
        f"Gold: {player.gold}\n"
        f"Reputation: {player.reputation}\n"
        f"Inventory: {inventory}\n"
        "----------------------"
    )


def ending(title: str, description: str) -> str:
    print_slow(f"ENDING: {title}")
    print(description)
    return "end"


def village_square(player: Player) -> str:
    print_slow(
        "The village of Brindlewick shivers beneath a red moon. "
        "Beyond the fields, the old Ashen Tower has awakened."
    )

    return ask_choice(
        "The elder begs you to investigate before dawn. Where do you go first?",
        {
            "1": ("Question the villagers at the tavern", "tavern"),
            "2": ("Take the forest road toward the tower", "forest"),
            "3": ("Visit the shrine beside the river", "shrine"),
            "4": ("Check your status", "status_village"),
        },
    )


def tavern(player: Player) -> str:
    print_slow(
        "The tavern is crowded with worried farmers, merchants, "
        "and one suspicious bard."
    )

    path = ask_choice(
        "Who do you approach?",
        {
            "1": ("Buy information from a merchant for 2 gold", "merchant"),
            "2": ("Help the bard calm the frightened crowd", "bard"),
            "3": ("Leave for the forest road", "forest"),
        },
    )

    if path == "merchant":
        if player.gold >= 2:
            player.gold -= 2
            player.has_ember_key = True
            player.inventory.append("Ember Key")
            print_slow("The merchant sells you the Ember Key.")
        else:
            print_slow("You do not have enough gold.")
        return "forest"

    if path == "bard":
        player.reputation += 2
        player.health += 1
        print_slow("You calm the crowd and gain reputation.")
        return "forest"

    return path


def shrine(player: Player) -> str:
    print_slow(
        "At the river shrine, silver reeds bow around a cracked statue. "
        "The water glows faintly."
    )

    path = ask_choice(
        "What do you do?",
        {
            "1": ("Drink the moonlit water", "drink"),
            "2": ("Offer 1 gold and fill a vial", "offer"),
            "3": ("Ignore the shrine and take the forest road", "forest"),
        },
    )

    if path == "drink":
        player.health += 3
        player.reputation -= 1
        print_slow("You feel stronger, but the shrine's glow fades.")
        return "forest"

    if path == "offer":
        if player.gold > 0:
            player.gold -= 1
            player.has_moon_water = True
            player.reputation += 1
            player.inventory.append("Moon Water")
            print_slow("You receive a vial of Moon Water.")
        else:
            print_slow("You have no gold to offer.")
        return "forest"

    return path


def forest(player: Player) -> str:
    print_slow(
        "The forest road twists under thorn and ash. "
        "A wounded wolf blocks the path."
    )

    path = ask_choice(
        "How do you handle the wolf?",
        {
            "1": ("Fight your way past", "fight_wolf"),
            "2": ("Use Moon Water to heal it", "heal_wolf"),
            "3": ("Sneak around through the brambles", "sneak"),
            "4": ("Check your status", "status_forest"),
        },
    )

    if path == "fight_wolf":
        player.health -= 4
        player.inventory.append("Wolf Fang")
        print_slow("You defeat the wolf, but you are injured.")

        if player.health <= 0:
            return ending(
                "Fallen on the Forest Road",
                "You fought bravely, but your journey ends in the forest.",
            )

        return "tower_gate"

    if path == "heal_wolf":
        if player.has_moon_water:
            player.has_moon_water = False
            player.inventory.remove("Moon Water")
            player.reputation += 2
            player.inventory.append("Wolf Companion")
            print_slow("The wolf is healed and becomes your ally.")
            return "tower_gate"

        player.health -= 2
        print_slow("You have no Moon Water. The wolf attacks before fleeing.")
        return "tower_gate"

    if path == "sneak":
        player.health -= 1
        player.gold += 2
        print_slow("You sneak through thorns and find a dropped purse.")
        return "tower_gate"

    return path


def tower_gate(player: Player) -> str:
    print_slow(
        "The Ashen Tower rises before you. "
        "Its gate has a keyhole shaped like a flame."
    )

    path = ask_choice(
        "How do you enter?",
        {
            "1": ("Use the Ember Key", "use_key"),
            "2": ("Force the gate open", "force_gate"),
            "3": ("Call out and ask the tower to let you in", "speak_gate"),
        },
    )

    if path == "use_key":
        if player.has_ember_key:
            print_slow("The Ember Key turns, and the gate opens.")
            return "tower_heart"

        player.health -= 2
        print_slow("You do not have the key. The tower burns you as it opens.")

        if player.health <= 0:
            return ending(
                "Claimed by the Lock",
                "The tower takes your life at the gate.",
            )

        return "tower_heart"

    if path == "force_gate":
        player.health -= 3
        print_slow("You force the gate open, but the effort wounds you.")

        if player.health <= 0:
            return ending(
                "Broken at the Threshold",
                "You open the tower, but cannot continue.",
            )

        return "tower_heart"

    if path == "speak_gate":
        if player.reputation >= 2:
            print_slow("The tower respects your deeds and opens.")
            return "tower_heart"

        return ending(
            "The Tower Refuses",
            "The tower rejects you, and your quest ends.",
        )

    return path


def tower_heart(player: Player) -> str:
    print_slow(
        "At the tower's heart, a crown of living flame floats above an empty throne."
    )

    path = ask_choice(
        "What is your final choice?",
        {
            "1": ("Destroy the crown", "destroy"),
            "2": ("Claim the crown", "claim"),
            "3": ("Bargain with the crown", "bargain"),
            "4": ("Check your status", "status_tower_heart"),
        },
    )

    if path == "destroy":
        if "Wolf Companion" in player.inventory or player.health >= 8:
            return ending(
                "Dawn Over Brindlewick",
                "You destroy the crown and save the village.",
            )

        return ending(
            "Victory at a Cost",
            "You destroy the crown, but sacrifice yourself to save Brindlewick.",
        )

    if path == "claim":
        if player.reputation < 0:
            return ending(
                "Ashen Tyrant",
                "The crown corrupts you, and you become the new threat.",
            )

        return ending(
            "The Watchful Monarch",
            "You claim the crown and protect Brindlewick from the tower.",
        )

    if path == "bargain":
        if player.has_ember_key and player.reputation >= 2:
            return ending(
                "Keeper of the Quiet Flame",
                "You make peace with the crown and bind its power safely.",
            )

        return ending(
            "A Poor Bargain",
            "The crown twists your wish, and the valley falls into endless sleep.",
        )

    return path


def play_game() -> None:
    print("Ashen Tower: A Text-Based RPG")
    print("=============================")

    name = input("Enter your hero's name: ").strip()

    if not name:
        name = "Wanderer"

    player = Player(name=name)

    scenes = {
        "village": village_square,
        "tavern": tavern,
        "shrine": shrine,
        "forest": forest,
        "tower_gate": tower_gate,
        "tower_heart": tower_heart,
    }

    current_scene = "village"

    while current_scene != "end":
        if current_scene.startswith("status_"):
            show_status(player)
            current_scene = current_scene.removeprefix("status_")
            continue

        scene = scenes[current_scene]
        current_scene = scene(player)

    print_slow("Thanks for playing.")


if __name__ == "__main__":
    play_game()
