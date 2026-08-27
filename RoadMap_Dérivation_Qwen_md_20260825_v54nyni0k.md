# SPEC — Application « ⛰️ DÉRIVATION » : généralisation & extension lycée général
# (dérivée de composée (u(v))′, exponentielle, trigonométrie)

Fichier cible : `1BP_Ma_Derivation_Sim_v2.html` (fichier unique, JS vanilla)
Périmètre    : Partie 2 (calcul de la dérivée) — extension filière générale +
               généralisation du moteur de niveaux/règles.
⚠️ Périmètre exclu : AUCUNE intégration avec `1CAP_Ma_Automatismes_Sim_v5.html`
   (la dérivation n'est pas au programme de CAP ; la sim CAP reste niveau collège).

---

## 1. État de l'art (existant)

| Partie | Contenu |
|---|---|
| 1. Lecture graphique | Montagne, promeneur draggable (🧗/⛷️/🧍), tangente colorée (vert/rouge/jaune), marche d'escalier (flèche +1 blanche, flèche hauteur rouge/verte), pente en fraction, tableau de variation à surlignage dynamique, mode paysage |
| 2. Calcul de f′ | Filières Bac Pro (niv 1–4 : polynômes) & Lycée général (niv 5–7 : u·v, u/v, uⁿ) ; pas à pas **2 colonnes (Calcul \| Règle)** ; parenthèses colorées par règle ; substitution animée (chips) ; quiz avec leurres ; gamification (★, 🔒, score, série) |
| 3. Calculatrice | Touches fonctionnelles AC / f(x) / f′(x) / SOLVE / SIGNE / VAR / EXE ; commandes « Dériver / résoudre » ; étude complète **sans discriminant** |

## 2. Objectifs

1. **Compléter la filière générale** : formes usuelles `(eˣ)′`, `(sin x)′`, `(cos x)′` ;
   règles `(eᵘ)′ = u′·eᵘ`, `(sin u)′ = u′·cos u`, `(cos u)′ = −u′·sin u` ;
   composée **`(u(v))′ = v′·(u′∘v)`**.
2. **Généraliser l'application** en moteur *data-driven* : registre de règles +
   tableau de niveaux ; ajouter un niveau = ajouter un objet de config, sans
   toucher au rendu.
3. Conserver : fichier unique, zéro dépendance, `cleanFloat`, virgule française,
   typographie façon LaTeX, a11y, responsive.

## 3. Registre des règles (à étendre)

| id | Formule affichée | Couleur (var CSS) | Filière |
|---|---|---|---|
| sum  | (u+v)′ = u′ + v′            | `--neon-purple`  | pro |
| k    | (k·u)′ = k·u′               | `--neon-orange`  | pro |
| pow  | (xⁿ)′ = n·xⁿ⁻¹              | `--neon-cyan`    | les deux |
| uv   | (u·v)′ = u′v + uv′          | `--neon-green`   | gen |
| quot | (u/v)′ = (u′v − uv′)/v²     | `--neon-magenta` | gen |
| compn| (uⁿ)′ = n·u′·uⁿ⁻¹           | `--neon-cyan`    | gen |
| expx | (eˣ)′ = eˣ                  | `--neon-lime`    | gen |
| expu | (eᵘ)′ = u′·eᵘ               | `--neon-lime`    | gen |
| sinx | (sin x)′ = cos x            | `--neon-pink`    | gen |
| cosx | (cos x)′ = −sin x           | `--neon-blue`    | gen |
| sinu | (sin u)′ = u′·cos u         | `--neon-pink`    | gen |
| cosu | (cos u)′ = −u′·sin u        | `--neon-blue`    | gen |
| chain| (u(v))′ = v′·(u′∘v)         | `--neon-teal`    | gen |

**Parenthèses colorées** : chaque id ↔ classe `p-{id}` ; les sous-expressions
concernées sont enveloppées `<b class="p-{id}">…</b>` ; le panneau latéral met
en `glow` la règle dont la ligne est active.

## 4. Modèle de données d'un niveau (généralisation)

~~~js
const LEVEL = {
  id:    Number,                 // 1..N
  track: "pro" | "gen",
  name:  String,                 // ex. "(ax+b)ⁿ", "e^{ax+b}"…
  rules: [ruleIds],              // règles mobilisées (panneau latéral)
  gen():  F,                     // générateur aléatoire → objet fonction F
  rows(F): Row[],                // [{calc, rule, hl}] — les 2 colonnes
  quiz(F): Choice[],             // [{html, key, good}] avec erreurs typiques
};
// Pipeline de rendu (inchangé) :
// renderLevels → newFunction → rowsFor → renderRows → activateRow(i)
// → highlightRule(hl) + animateChips ; quiz : buildQuiz → onQuiz → correction auto.
~~~

=> **Ajouter un niveau = pousser un objet dans `LEVELS`.** Le registre `RULES`
   centralise formules + couleurs (une seule déclaration, trois usages :
   parenthèses, colonne Règle, panneau latéral).

## 5. Nouveaux niveaux (filière générale)

### Niveau 8 — Exponentielle
- Formes usuelles : `(eˣ)′ = eˣ` ; puis `(eᵘ)′ = u′·eᵘ` (u affine, puis u polynôme degré 2).
- Générateur : u = [a,b], a∈[1,3], b∈[−4,4] ; variante u = [1,b,c].
- Erreurs typiques (quiz) : `eᵘ` (oubli de u′) ; `a·eᵃ` (exposant dérivé) ; `a·x·eᵘ`.

### Niveau 9 — Trigonométrie
- Formes usuelles : `(sin x)′ = cos x` ; `(cos x)′ = −sin x`.
- Règles : `(sin u)′ = u′·cos u` ; `(cos u)′ = −u′·sin u` (u affine puis x²).
- Erreurs typiques : `cos u` sans u′ ; `−cos u` (mauvais signe) ; `u′·sin u`
  (confusion sin/cos) ; perte du terme constant.

### Niveau 10 — Composée `(u(v))′`
- Énoncé de la règle : **`(u∘v)′(x) = v′(x)·u′(v(x))`**
  (« dérivée de l'externe évaluée en l'interne × dérivée de l'interne »).
- Externe ∈ {Xⁿ, eˣ, sin X, cos X} ; interne ∈ {affine, trinôme simple}.
- Étapes : identifier v (interne) et u (externe) → v′ → u′ → assembler `v′·(u′∘v)`.
- Erreurs typiques : oubli de v′ ; `u(v′)` ; `u′·v′` ; `u′(x)·v(x)`.

## 6. Exemples de rendu 2 colonnes (référence à reproduire)

**Niv 8 — f(x) = e²ˣ⁺¹**
| Calcul | Règle appliquée |
|---|---|
| f(x) = e^(2x+1) | fonction à dériver : eᵘ |
| u = 2x+1 → u′ = 2 | (xⁿ)′ = n·xⁿ⁻¹ |
| f′(x) = u′·eᵘ | (eᵘ)′ = u′·eᵘ |
| f′(x) = (2)·e^(2x+1) | substitution |
| f′(x) = 2e^(2x+1) | simplification |

**Niv 9 — f(x) = sin(3x−1)** → f′(x) = 3·cos(3x−1) (règle sinu)
**Niv 9 — f(x) = cos(2x)**   → f′(x) = −2·sin(2x) (règle cosu)
**Niv 10 — f(x) = e^(x²+1)** : v = x²+1, v′ = 2x ; u = eˣ, u′ = eˣ
→ f′(x) = 2x·e^(x²+1) (règle chain)

## 7. Gamification (inchangée, étendue aux niv 8–10)
3 ★ par niveau ; niveau suivant débloqué à 3 ★ ; score += 10 × niveau ;
série 🔥 ; confettis au déblocage. Chaîne gen : 5 → 6 → 7 → 8 → 9 → 10.

## 8. Rendu & typographie
- Variables italiques, exposants `<sup>`, fractions empilées `.frac`.
- Exponentielle : `e<sup>…</sup>` (HTML) ; `exp(…)` sur l'écran LCD (partie 3).
- Virgule décimale française ; `cleanFloat` sur tout calcul intermédiaire.
- Panneau latéral : groupe « Lycée général » enrichi des nouvelles formules.

## 9. Robustesse (reprise des SPECS)
`cleanFloat` (IEEE 754) ; sanitisation des saisies (pas de NaN) ; SVG responsive
(`viewBox`, `width:100%`) ; a11y (tabindex, flèches clavier, aria) ;
Pointer Events ; fichier unique.

## 10. Matrice de tests (extraits)
| ID | Niveau | Action | Résultat attendu |
|---|---|---|---|
| TC08 | 8 | pas à pas e^(2x+1) | 5 lignes ; règle expu en glow ; final 2e^(2x+1) |
| TC09 | 8 | quiz | leurre e^(2x+1) marqué ❌ ; bonne réponse 2e^(2x+1) |
| TC10 | 9 | sin(3x−1) | final 3cos(3x−1) ; règle sinu en glow |
| TC11 | 9 | cos(2x) | final −2sin(2x) |
| TC12 | 10 | e^(x²+1) | final 2x·e^(x²+1) ; règle chain en glow |
| TC13 | 10 | quiz | leurre sans v′ marqué ❌ |
| TC14 | global | accès niv 9 sans 3★ au niv 8 | bouton 🔒 désactivé |

## 11. PROMPT réutilisable (génération d'un niveau)

~~~md
## PROMPT — Ajouter le niveau {N} « {NOM} » à l'application ⛰️ DÉRIVATION

Contexte : app fichier unique (JS vanilla) « ⛰️ DÉRIVATION », partie 2.
Moteur data-driven : registre RULES (formule + couleur) + tableau LEVELS
(gen / rows / quiz). Rendu : pas à pas 2 colonnes (Calcul | Règle),
parenthèses colorées p-{id}, glow du panneau latéral, chips de substitution,
quiz à leurres, gamification ★/🔒/score/série.

Ajoute le niveau {N} (filière {TRACK}) :
1. Fonction : {description + générateur aléatoire}
2. Règles : {ids} — si nouvelle : registre + couleur dédiée + formule panneau.
3. Étapes 2 colonnes : {liste des lignes calc/règle} avec parenthèses colorées.
4. Quiz : bonne réponse + 3 erreurs typiques {liste}.
5. Verrou : 3 ★ au niveau {N−1}.

Contraintes : fichier .html unique, sans librarie ; cleanFloat ; virgule
française ; typographie LaTeX-like (italique, sup, .frac) ; a11y clavier ;
responsive ; AUCUNE intégration avec la sim CAP.

Livrable : fichier .html unique mis à jour + lignes de matrice de tests.
~~~

## 12. Roadmap
1. Refactor partie 2 en moteur `RULES + LEVELS` (comportement inchangé).
2. Étendre registre + panneau : expx, expu, sinx, cosx, sinu, cosu, chain.
3. Ajouter niveaux 8, 9, 10 (rows + quiz).
4. Dérouler la matrice de tests ; valider module par module.
5. (Optionnel, hors périmètre actuel) ln : (ln x)′ = 1/x ; (ln u)′ = u′/u.