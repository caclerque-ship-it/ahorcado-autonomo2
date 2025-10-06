# ahorcado-autonomo2
Juego Ahorcado
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Juego: El Ahorcado (consola)
Archivo: ahorcado.py

Características:
- Selección aleatoria de palabra desde una lista interna o desde 'words.txt' si existe.
- Manejo de intentos y estados del "ahorcado" (dibujos ASCII).
- Validación de entrada (solo letras, no repetir intentos).
- Comentarios en las secciones complejas para facilitar entendimiento.
- Uso de estructuras repetitivas y condicionales (while, for, if/else).
- Extensible: puedes agregar más palabras en 'words.txt', una por línea.
"""

import random
import os
import sys
import string
from typing import List, Set

# --- Configuración: lista de palabras por defecto ---
DEFAULT_WORDS = [
    "python", "desarrollo", "programacion", "computadora", "algoritmo",
    "variable", "funcion", "estructura", "iteracion", "condicional"
]

# --- Estados del dibujo del ahorcado (de 0 errores hasta N errores) ---
HANGMAN_STAGES = [
    # 0 errores
    """
     _______
    |/      |
    |
    |
    |
    |
    |
    |___
    """,
    # 1 error
    """
     _______
    |/      |
    |      (_)
    |
    |
    |
    |
    |___
    """,
    # 2 errores
    """
     _______
    |/      |
    |      (_)
    |       |
    |       |
    |
    |
    |___
    """,
    # 3 errores
    """
     _______
    |/      |
    |      (_)
    |      \\|
    |       |
    |
    |
    |___
    """,
    # 4 errores
    """
     _______
    |/      |
    |      (_)
    |      \\|/
    |       |
    |
    |
    |___
    """,
    # 5 errores
    """
     _______
    |/      |
    |      (_)
    |      \\|/
    |       |
    |      /
    |
    |___
    """,
    # 6 errores -> completo (derrota)
    """
     _______
    |/      |
    |      (_)
    |      \\|/
    |       |
    |      / \\
    |
    |___
    """
]

MAX_ERRORS = len(HANGMAN_STAGES) - 1  # número máximo de errores permitidos

# --- Utilidades ---


def clear_screen():
    """Limpia la pantalla de la consola (compatible con Windows y Unix)."""
    os.system("cls" if os.name == "nt" else "clear")


def load_words_from_file(filename: str) -> List[str]:
    """
    Intenta cargar una lista de palabras desde un archivo.
    Si falla, retorna la lista por defecto.
    """
    try:
        with open(filename, "r", encoding="utf-8") as f:
            words = [line.strip() for line in f if line.strip()]
        # filtrar palabras que contengan solo letras
        words = [w.lower() for w in words if all(ch in string.ascii_letters for ch in w.lower())]
        if words:
            return words
        else:
            return DEFAULT_WORDS
    except FileNotFoundError:
        return DEFAULT_WORDS
    except Exception:
        # En caso de cualquier otro error, usar por defecto
        return DEFAULT_WORDS


def choose_word(words: List[str]) -> str:
    """Selecciona aleatoriamente una palabra de la lista dada."""
    return random.choice(words).lower()


def display_game_state(secret_word: str, guessed_letters: Set[str], errors: int):
    """
    Muestra en pantalla:
    - Dibujo del ahorcado según la cantidad de errores.
    - Palabra oculta con letras adivinadas.
    - Letras ya intentadas (separadas entre correctas e incorrectas).
    """
    print(HANGMAN_STAGES[errors])
    # Generar la representación de la palabra (ej: p _ t h o n)
    shown = " ".join([ch if ch in guessed_letters else "_" for ch in secret_word])
    print("Palabra: ", shown)
    # Letras intentadas
    correct = sorted([c for c in guessed_letters if c in secret_word])
    incorrect = sorted([c for c in guessed_letters if c not in secret_word])
    print(f"Letras correctas: {', '.join(correct) if correct else '---'}")
    print(f"Letras incorrectas: {', '.join(incorrect) if incorrect else '---'}")
    print(f"Errores: {errors}/{MAX_ERRORS}")
    print()


def get_user_input(already_guessed: Set[str]) -> str:
    """
    Solicita al usuario una letra válida:
    - Debe ser una sola letra del alfabeto (no números ni símbolos).
    - No debe haber sido intentada previamente.
    """
    while True:
        guess = input("Ingresa una letra: ").strip().lower()
        if len(guess) != 1:
            print("Por favor ingresa solo UNA letra.")
            continue
        if guess not in string.ascii_lowercase:
            print("Por favor ingresa una letra válida (a-z).")
            continue
        if guess in already_guessed:
            print("Ya intentaste esa letra. Prueba con otra.")
            continue
        return guess


def has_won(secret_word: str, guessed_letters: Set[str]) -> bool:
    """Devuelve True si todas las letras de la palabra han sido adivinadas."""
    # Usamos set para ignorar repeticiones en la palabra secreta
    return set(secret_word).issubset(guessed_letters)


# --- Lógica principal del juego ---


def play_round(words: List[str]):
    """
    Ejecuta una ronda completa del juego (hasta victoria o derrota).
    Esta función contiene estructuras repetitivas para el flujo principal.
    """
    secret_word = choose_word(words)
    guessed_letters: Set[str] = set()
    errors = 0

    # Bucle principal del juego: sigue hasta que gane o alcance MAX_ERRORS
    while True:
        clear_screen()
        display_game_state(secret_word, guessed_letters, errors)

        # Comprobar condiciones de fin antes de pedir nueva entrada
        if has_won(secret_word, guessed_letters):
            print("¡Felicidades! ¡Has adivinado la palabra!")
            print(f"La palabra era: {secret_word}")
            break
        if errors >= MAX_ERRORS:
            print("Has alcanzado el número máximo de errores. ¡Has perdido!")
            print(f"La palabra era: {secret_word}")
            break

        # Solicitar letra al usuario (validación incluida)
        guess = get_user_input(guessed_letters)

        # Verificar si la letra está en la palabra
        if guess in secret_word:
            # Añadir a letras adivinadas (correctas)
            guessed_letters.add(guess)
            print(f"¡Bien! La letra '{guess}' está en la palabra.")
            # pequeña pausa para que el usuario vea el mensaje
            input("Presiona Enter para continuar...")
        else:
            # Añadir a letras intentadas (incorrectas) y sumar error
            guessed_letters.add(guess)
            errors += 1
            print(f"Lo siento. La letra '{guess}' NO está en la palabra.")
            input("Presiona Enter para continuar...")


def main():
    """Función principal: menú y control de múltiples partidas."""
    words = load_words_from_file("words.txt")  # si existe, cargará palabras adicionales
    random.seed()  # inicializa la semilla aleatoria

    while True:
        clear_screen()
        print("=== JUEGO: EL AHORCADO ===")
        print("1) Jugar")
        print("2) Ver lista de palabras (muestra algunas)")
        print("3) Agregar una palabra al archivo 'words.txt' (opcional)")
        print("4) Salir")
        choice = input("Elige una opción (1-4): ").strip()

        if choice == "1":
            play_round(words)
            # Al terminar la ronda, preguntar si desea jugar otra vez
            again = input("¿Quieres jugar otra vez? (s/n): ").strip().lower()
            if again not in ("s", "si", "y", "yes"):
                print("Gracias por jugar. ¡Hasta luego!")
                break
        elif choice == "2":
            clear_screen()
            print("Muestra de palabras disponibles (máx 20):")
            # imprimir un subconjunto al azar para evitar mostrar listas largas completas
            sample = random.sample(words, min(20, len(words)))
            print(", ".join(sample))
            input("\nPresiona Enter para volver al menú...")
        elif choice == "3":
            # Opción para agregar nuevas palabras al archivo words.txt
            new_word = input("Ingresa la palabra a agregar (solo letras): ").strip().lower()
            if not new_word or any(ch not in string.ascii_lowercase for ch in new_word):
                print("Palabra inválida. Debe contener solo letras sin espacios ni números.")
                input("Presiona Enter para volver al menú...")
                continue
            try:
                # Apendeamos al archivo (lo crea si no existe)
                with open("words.txt", "a", encoding="utf-8") as f:
                    f.write(new_word + "\n")
                # También actualizar la lista en memoria para usarla inmediatamente
                words.append(new_word)
                print(f"Palabra '{new_word}' agregada correctamente a words.txt.")
            except Exception as e:
                print("Ocurrió un error al escribir en 'words.txt':", e)
            input("Presiona Enter para volver al menú...")
        elif choice == "4":
            print("Saliendo... ¡Gracias por jugar!")
            break
        else:
            print("Opción no válida. Intenta nuevamente.")
            input("Presiona Enter para volver al menú...")


if __name__ == "__main__":
    main()
