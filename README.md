# contaminacio-luminica-lux-analyzer
Programa en Python per al tractament de mesures de luminància (lux) en un estudi de contaminació lumínica. Calcula la mitjana, l'error experimental, combina l'error instrumental i ofereix un resultat final arrodonit segons xifres significatives.
"""
Programa per al tractament de mesures de luminància (lux).

Objectiu:
    - Llegir un conjunt de mesures de luminància (en lux).
    - Calcular la mitjana.
    - Calcular la desviació estàndard de la mitjana (incertesa experimental).
    - Combinar-la amb la incertesa instrumental.
    - Arrodonir automàticament valor i error segons xifres significatives.
    - Mostrar el resultat final com: X = (valor ± error) lux.

Autor: Amira
Llenguatge: Python 3.12
Llibreria usada: math
Unitats: lux
σ_instr (incertesa instrumental estàndard): 0.1 lux
"""

import math

# Incertesa instrumental (es pot modificar segons l'instrument real)
SIGMA_INSTR = 0.1  # lux


def llegir_nombre(missatge: str) -> float:
    """Llegeix un nombre en coma flotant de l'usuari amb control d'errors."""
    while True:
        entrada = input(missatge).strip()
        try:
            valor = float(entrada)
            return valor
        except ValueError:
            print("❌ Si us plau, introdueix un valor numèric vàlid.")


def llegir_enter_positiu(missatge: str) -> int:
    """Llegeix un enter positiu (N de mesures)."""
    while True:
        entrada = input(missatge).strip()
        try:
            n = int(entrada)
            if n > 0:
                return n
            else:
                print("❌ El nombre de mesures ha de ser un enter positiu.")
        except ValueError:
            print("❌ Si us plau, introdueix un nombre enter vàlid.")


def llegir_mesures(n: int) -> list[float]:
    """Llegeix N mesures de luminància en lux."""
    mesures = []
    print(f"\nIntrodueix les {n} mesures en lux:")
    for i in range(1, n + 1):
        valor = llegir_nombre(f"  Mesura {i}: ")
        mesures.append(valor)
    return mesures


def calcular_mitjana(mesures: list[float]) -> float:
    """Calcula la mitjana de les mesures."""
    return sum(mesures) / len(mesures)


def calcular_desviacio_estandard(mesures: list[float], mitjana: float) -> float:
    """Calcula la desviació estàndard de les mesures (mostral, N-1)."""
    n = len(mesures)
    if n < 2:
        return 0.0
    suma = sum((x - mitjana) ** 2 for x in mesures)
    return math.sqrt(suma / (n - 1))


def calcular_incertesa_experimental_mesura(mesures: list[float]) -> float:
    """
    Calcula la incertesa experimental de la mitjana:
    σ_exp = s / sqrt(N), on s és la desviació estàndard mostral.
    """
    n = len(mesures)
    mitjana = calcular_mitjana(mesures)
    s = calcular_desviacio_estandard(mesures, mitjana)
    if n == 0:
        return 0.0
    return s / math.sqrt(n)


def combinar_incerteses(sigma_exp: float, sigma_instr: float) -> float:
    """
    Combina la incertesa experimental amb la instrumental:
    σ_total = sqrt(σ_exp^2 + σ_instr^2)
    """
    return math.sqrt(sigma_exp**2 + sigma_instr**2)


def arrodonir_valor_i_error(valor: float, error: float) -> tuple[float, float]:
    """
    Arrodoneix valor i error segons l'ordre de magnitud de l'error.

    Estratègia:
      - Es deixa l'error amb 1 xifra significativa.
      - El valor es redueix als mateixos decimals que l'error.
    """
    if error == 0:
        # Cas patològic: totes les mesures iguals i sense incertesa instrumental
        return round(valor, 3), 0.0

    # Magnitud de l'error (exponent en base 10)
    exponent = math.floor(math.log10(abs(error)))
    # Error amb 1 xifra significativa
    factor = 10 ** exponent
    primer_digit = error / factor
    error_arrodonit = round(primer_digit) * factor

    # Nombre de decimals a mostrar
    # Exemple: error = 0.12  -> exponent = -1  -> decimals = 1
    #          error = 0.007 -> exponent = -3  -> decimals = 3
    if error_arrodonit != 0:
        decimals = max(0, -math.floor(math.log10(abs(error_arrodonit))))
    else:
        decimals = 3

    valor_arrodonit = round(valor, decimals)
    error_arrodonit = round(error_arrodonit, decimals)

    return valor_arrodonit, error_arrodonit


def main():
    print("=== Programa de tractament de mesures de luminància (lux) ===\n")

    # 1. Entrada del nombre de mesures
    n = llegir_enter_positiu("Introdueix el nombre de mesures (N): ")

    # 2. Entrada de les mesures
    mesures = llegir_mesures(n)

    # 3. Càlcul de la mitjana
    mitjana = calcular_mitjana(mesures)

    # 4. Incertesa experimental de la mitjana
    sigma_exp = calcular_incertesa_experimental_mesura(mesures)

    # 5. Combinació amb la incertesa instrumental
    sigma_total = combinar_incerteses(sigma_exp, SIGMA_INSTR)

    # 6. Arrodoniment segons l'ordre de magnitud de l'error
    valor_final, error_final = arrodonir_valor_i_error(mitjana, sigma_total)

    # 7. Presentació de resultats
    print("\n--- RESULTATS ---")
    print(f"Nombre de mesures (N): {n}")
    print(f"Mitjana (no arrodonida): {mitjana:.6f} lux")
    print(f"Incertesa experimental de la mitjana: {sigma_exp:.6f} lux")
    print(f"Incertesa instrumental: {SIGMA_INSTR:.6f} lux")
    print(f"Incertesa total (no arrodonida): {sigma_total:.6f} lux")

    print("\nResultat final:")
    print(f"X = ({valor_final} ± {error_final}) lux")


if __name__ == "__main__":
    main()
