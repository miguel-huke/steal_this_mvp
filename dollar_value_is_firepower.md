#!/usr/bin/env python3
# reserve_currency_sim.py
"""
Simulador educativo inspirado na frase "dólar é lastreado em poder de fogo".

Ideia: a atratividade de uma moeda de reserva depende de múltiplos pilares:
- instituições/estado de direito (rule_of_law)
- profundidade e liquidez de mercados (market_depth)
- estabilidade macro (macro_stability)
- rede de alianças e integração (alliances_network)
- capacidade de projeção de poder/segurança (security_umbrella)  <-- abstração
- tecnologia/soft power (soft_power)
- risco de sanções / uso político do sistema financeiro (sanctions_risk)
- confiança internacional (trust)

O modelo é heurístico e serve apenas para explorar cenários.
"""

from __future__ import annotations
from dataclasses import dataclass
from math import exp
import json
import random
import argparse
from typing import Dict, List, Tuple

# -------------------------
# Utilitários
# -------------------------

def clamp(x: float, lo: float = 0.0, hi: float = 1.0) -> float:
    return max(lo, min(hi, x))

def sigmoid(x: float) -> float:
    # converte valores reais em 0..1
    return 1.0 / (1.0 + exp(-x))

def weighted_sum(features: Dict[str, float], weights: Dict[str, float]) -> float:
    s = 0.0
    for k, w in weights.items():
        s += w * features.get(k, 0.0)
    return s

# -------------------------
# Modelo
# -------------------------

@dataclass
class ReserveCurrencyInputs:
    rule_of_law: float         # 0..1
    market_depth: float        # 0..1
    macro_stability: float     # 0..1
    alliances_network: float   # 0..1
    security_umbrella: float   # 0..1 (abstração de garantia/segurança)
    soft_power: float          # 0..1
    trust: float               # 0..1

    sanctions_risk: float      # 0..1 (quanto maior, pior para a neutralidade)
    political_volatility: float# 0..1 (quanto maior, pior)

    def as_dict(self) -> Dict[str, float]:
        return {
            "rule_of_law": self.rule_of_law,
            "market_depth": self.market_depth,
            "macro_stability": self.macro_stability,
            "alliances_network": self.alliances_network,
            "security_umbrella": self.security_umbrella,
            "soft_power": self.soft_power,
            "trust": self.trust,
            "sanctions_risk": self.sanctions_risk,
            "political_volatility": self.political_volatility,
        }

DEFAULT_WEIGHTS = {
    # pilares positivos
    "rule_of_law": 1.30,
    "market_depth": 1.45,
    "macro_stability": 1.25,
    "alliances_network": 1.00,
    "security_umbrella": 0.85,
    "soft_power": 0.70,
    "trust": 1.20,
    # fricções (negativas)
    "sanctions_risk": -0.95,
    "political_volatility": -1.00,
}

def reserve_attractiveness(inp: ReserveCurrencyInputs, weights: Dict[str, float] = DEFAULT_WEIGHTS) -> float:
    """
    Retorna 0..1: quão atrativa é a moeda como reserva internacional.
    """
    x = weighted_sum(inp.as_dict(), weights)

    # Interações simples: confiança amplifica liquidez e instituições;
    # risco de sanções reduz confiança efetiva (neutralidade percebida).
    trust_effective = clamp(inp.trust * (1.0 - 0.55 * inp.sanctions_risk))
    x += 0.60 * (inp.market_depth * trust_effective)
    x += 0.45 * (inp.rule_of_law * trust_effective)

    # Estabilidade macro perde muito com volatilidade política
    x += 0.40 * (inp.macro_stability * (1.0 - inp.political_volatility))

    # Normaliza para 0..1
    return clamp(sigmoid(x - 2.5))  # deslocamento para calibrar

def hegemony_index(inp: ReserveCurrencyInputs) -> float:
    """
    0..1: um índice “narrativo” de hegemonia percebida (não é PIB real).
    Pesa mais rede + segurança + mercados.
    """
    base = (
        0.25 * inp.market_depth +
        0.20 * inp.alliances_network +
        0.20 * inp.security_umbrella +
        0.15 * inp.soft_power +
        0.10 * inp.rule_of_law +
        0.10 * inp.trust
    )
    friction = 0.20 * inp.political_volatility + 0.15 * inp.sanctions_risk
    return clamp(base - friction)

# -------------------------
# Cenários
# -------------------------

PRESETS: Dict[str, ReserveCurrencyInputs] = {
    # Esses presets são "arquétipos" abstratos (não países explicitamente).
    "hegemon": ReserveCurrencyInputs(
        rule_of_law=0.75, market_depth=0.95, macro_stability=0.70,
        alliances_network=0.85, security_umbrella=0.90, soft_power=0.80,
        trust=0.78,
        sanctions_risk=0.55, political_volatility=0.45
    ),
    "neutral_hub": ReserveCurrencyInputs(
        rule_of_law=0.90, market_depth=0.70, macro_stability=0.85,
        alliances_network=0.55, security_umbrella=0.35, soft_power=0.60,
        trust=0.88,
        sanctions_risk=0.15, political_volatility=0.20
    ),
    "regional_power": ReserveCurrencyInputs(
        rule_of_law=0.60, market_depth=0.60, macro_stability=0.55,
        alliances_network=0.60, security_umbrella=0.65, soft_power=0.55,
        trust=0.55,
        sanctions_risk=0.35, political_volatility=0.40
    ),
}

SHOCKS: Dict[str, Dict[str, float]] = {
    # mudanças delta (podem ser negativas ou positivas)
    "financial_crisis": {"macro_stability": -0.25, "trust": -0.18, "market_depth": -0.10},
    "institutional_reform": {"rule_of_law": +0.18, "trust": +0.12, "political_volatility": -0.10},
    "sanctions_wave": {"sanctions_risk": +0.25, "trust": -0.15, "alliances_network": -0.05},
    "security_pact": {"alliances_network": +0.15, "security_umbrella": +0.18, "trust": +0.05},
    "polarization": {"political_volatility": +0.20, "trust": -0.12},
    "tech_boom": {"soft_power": +0.18, "market_depth": +0.08, "trust": +0.05},
}

def apply_shock(inp: ReserveCurrencyInputs, shock: Dict[str, float]) -> ReserveCurrencyInputs:
    d = inp.as_dict()
    for k, delta in shock.items():
        d[k] = clamp(d[k] + delta)
    return ReserveCurrencyInputs(**d)  # type: ignore

def simulate(
    start: ReserveCurrencyInputs,
    steps: int = 12,
    seed: int | None = None,
    random_noise: float = 0.03,
    shock_sequence: List[str] | None = None,
) -> List[Tuple[int, ReserveCurrencyInputs, float, float]]:
    """
    Retorna lista de (t, inputs, attract, hegemony).
    """
    if seed is not None:
        random.seed(seed)

    cur = start
    out = []
    for t in range(steps + 1):
        att = reserve_attractiveness(cur)
        heg = hegemony_index(cur)
        out.append((t, cur, att, heg))

        # aplica choque do passo seguinte (se existir)
        if t == steps:
            break

        nxt = cur
        if shock_sequence and t < len(shock_sequence):
            name = shock_sequence[t]
            if name and name in SHOCKS:
                nxt = apply_shock(nxt, SHOCKS[name])

        # ruído pequeno (representa “fricções” e aleatoriedade)
        d = nxt.as_dict()
        for k in d.keys():
            if k in ("sanctions_risk", "political_volatility"):
                # fricções também variam, mas menos
                d[k] = clamp(d[k] + random.uniform(-random_noise, random_noise) * 0.6)
            else:
                d[k] = clamp(d[k] + random.uniform(-random_noise, random_noise))
        cur = ReserveCurrencyInputs(**d)  # type: ignore

    return out

def print_report(rows: List[Tuple[int, ReserveCurrencyInputs, float, float]]) -> None:
    print("\n=== Reserve Currency Simulator ===\n")
    print("t | attract | hegemony | key notes")
    print("--+---------+----------+------------------------------")
    for t, inp, att, heg in rows:
        note = []
        if inp.sanctions_risk > 0.6: note.append("sanctions-risk high")
        if inp.market_depth > 0.85: note.append("deep markets")
        if inp.rule_of_law > 0.85: note.append("strong institutions")
        if inp.political_volatility > 0.6: note.append("volatile politics")
        if inp.security_umbrella > 0.8: note.append("security umbrella strong")
        msg = ", ".join(note) if note else "-"
        print(f"{t:>2} |  {att:>0.3f}  |  {heg:>0.3f}   | {msg}")

    final_att = rows[-1][2]
    final_heg = rows[-1][3]
    print("\nInterpretation:")
    print(f"- Final attractiveness: {final_att:.3f} (0..1)")
    print(f"- Final hegemony index: {final_heg:.3f} (0..1)")
    print("- Use os shocks para testar a hipótese: “poder/segurança” ajuda, mas pode ser anulado por")
    print("  risco político, uso de sanções e queda de confiança.\n")

def to_json(rows: List[Tuple[int, ReserveCurrencyInputs, float, float]]) -> Dict:
    return {
        "series": [
            {
                "t": t,
                "inputs": inp.as_dict(),
                "attractiveness": att,
                "hegemony_index": heg,
            }
            for t, inp, att, heg in rows
        ]
    }

# -------------------------
# CLI
# -------------------------

def main():
    p = argparse.ArgumentParser(description="Simulador educativo: moeda de reserva vs. poder/segurança e confiança.")
    p.add_argument("--preset", choices=PRESETS.keys(), default="hegemon", help="Arquétipo inicial.")
    p.add_argument("--steps", type=int, default=12, help="Número de passos (ex: meses/anos).")
    p.add_argument("--seed", type=int, default=7, help="Seed aleatória.")
    p.add_argument("--noise", type=float, default=0.03, help="Ruído (0..0.1 típico).")
    p.add_argument("--shocks", type=str, default="", help="Sequência CSV de choques (ex: tech_boom,polarization,sanctions_wave).")
    p.add_argument("--export", type=str, default="", help="Exporta JSON para este arquivo.")
    p.add_argument("--list-shocks", action="store_true", help="Lista choques disponíveis.")
    args = p.parse_args()

    if args.list_shocks:
        print("Available shocks:")
        for k in SHOCKS.keys():
            print(" -", k, SHOCKS[k])
        return

    start = PRESETS[args.preset]
    shock_seq = [s.strip() for s in args.shocks.split(",") if s.strip()] if args.shocks else None

    rows = simulate(
        start=start,
        steps=args.steps,
        seed=args.seed,
        random_noise=args.noise,
        shock_sequence=shock_seq,
    )

    print_report(rows)

    if args.export:
        data = to_json(rows)
        with open(args.export, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
        print(f"Exported JSON to: {args.export}")

if __name__ == "__main__":
    main()
