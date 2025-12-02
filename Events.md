

# **Les Events dans Unreal Engine**


# INTRODUCTION — Le rôle des Events dans Unreal Engine

Les Events permettent à Unreal Engine de réagir à toute action ou changement d’état :

- Touche pressée
- Collision
- Animation
- Timer
- UI
- Spawn/destruction d’acteur
- Mouvement continu

Tout gameplay repose sur eux.

---

# I — Events du Cycle de Vie (Les Fondamentaux)

## 1. Event BeginPlay

###  Explication
S'exécute une seule fois : au lancement du jeu ou lors du spawn de l’acteur.

### Utilisations
- Initialisation
- Timers
- Sons et musiques
- Spawn d’objets/ennemis
- Ouverture de menus

### Exemple
```blueprint
Event BeginPlay
    → PrintString("Démarrage du jeu")
````

### Exemple avancé

```blueprint
Event BeginPlay
    → Delay 2s
    → SpawnActor(EnemyBP)
```

### Exercice 1

Créer un acteur qui spawn **5 cubes** espacés de **200 unités** au début du jeu.

---

## 2. Event Tick

### Explication

Exécuté chaque frame (30-120 fois/sec selon FPS).

### Bon usages

* Mouvements continus
* Oscillations (sinus, cosinus)
* Effets visuels dynamiques

### Exemple : rotation

```blueprint
Event Tick
   → AddActorLocalRotation (Yaw = 80 * DeltaSeconds)
```

### Exemple : oscillation

```blueprint
Event Tick
    → NewZ = Sin(Time) * Amplitude
    → SetActorLocation
```

### Exercice 2

Créer un acteur qui flotte avec sinus (Amplitude + Speed exposées).

---

## 3. Event EndPlay

### Explication

Déclenché lorsque l’acteur est détruit ou le niveau change.

### Exemple

```blueprint
Event EndPlay
    → ClearTimer(MyTimer)
```

### Exercice 3

Un acteur qui :

* démarre un Timer en BeginPlay
* affiche “Tick…” toutes les 0.5s
* arrête le Timer dans EndPlay

---

## 4. Construction Script

### Explication

Exécuté **dans l’éditeur**, avant le jeu.

### Utilité

* Préparer les meshes
* Génération procédurale
* Mise à jour visuelle dynamique

---

# II — Events de Collision

Deux familles :

1. Overlap → Zones / interactions
2. Hit → Collisions physiques réelles

---

# 1. Overlap Events

## OnComponentBeginOverlap

### Utilité

Détection d’entrée dans une zone.

### Exemple : pickup

```blueprint
OnBeginOverlap
    → Cast OtherActor to Player
    → AddItem
    → DestroyActor(self)
```

### Exemple : ouvrir une porte

```blueprint
OnBeginOverlap
   → PlayTimeline(OpenDoor)
```

---

## OnComponentEndOverlap

### Exemple

```blueprint
OnEndOverlap
   → HideWidget("Press E")
```

---

### Exercice 4

Créer une **zone de soin** :

* BeginOverlap → Timer → +10 HP toutes les 2s
* EndOverlap → stop Timer
* max HP = 100

---

# 2. Hit Events

Déclenchés lors de collisions physiques.

### Exemple : explosion d’un projectile

```blueprint
Event Hit
    → SpawnEmitterAtLocation(Explosion)
    → ApplyDamage
    → DestroyActor(self)
```

### Exemple : rebond

```blueprint
Event Hit
    → AddImpulse (HitNormal * ReboundForce)
```

### Exercice 5

Créer une boule qui :

* SimulatePhysics ON
* joue un son au contact
* rebondit avec AddImpulse

---

# III — Events d’Entrée (Enhanced Input)

## 1. Action Events (Press / Release)

### Exemple : sauter

```blueprint
IA_Jump (Triggered)
   → Jump
```

### Exemple : toggle lumière

```blueprint
IA_ToggleLight Triggered
    → FlipFlop
```

### Exercice 6

Créer un Dash :

* vitesse *augmentée* pendant 0.4s
* cooldown 1.5s
* effet visuel (Niagara)

---

## 2. Axis Events (valeurs continues)

### Exemple : avancer/reculer

```blueprint
IA_MoveForward (AxisValue)
     → AddMovementInput(ForwardVector, AxisValue)
```

### Exemple : rotation caméra

```blueprint
CameraYaw += AxisValue * Sensitivity
```

### 🏋️‍♂️ Exercice 7

Créer un système caméra :

* IA_Turn
* IA_LookUp
* Sensibilité réglable via Slider UI

---

# IV — UI Events (UMG)

## 1. Boutons

* OnClicked
* OnHovered
* OnPressed
* OnReleased

### Exemple

```blueprint
OnClicked(ButtonStart)
     → OpenLevel("MainMap")
```

---

## 2. Sliders

### Exemple

```blueprint
OnValueChanged(SliderVolume)
    → SetVolume(SliderValue)
```

---

## 3. TextBox

```blueprint
OnTextChanged(TextBoxName)
    → PlayerName = NewText
```

---

## 4. ListView

### Exemple

```blueprint
OnItemClicked
    → DisplayDescription
```

---

### Exercice 8

Créer un menu :

* Slider → change la vitesse du joueur
* TextBox → renomme le joueur
* Bouton → réinitialise la vitesse

---

# V — Event Dispatchers (Signaux)

### Explication

Permet à un acteur *A* de notifier *B* sans dépendance directe.

---

## Exemple complet : mise à jour de la vie

### 1. Déclarer le dispatcher dans Player

```
OnHealthChanged(NewHealth)
```

### 2. L’appeler

```blueprint
TakeDamage
    → Health -= Damage
    → OnHealthChanged.Broadcast(Health)
```

### 3. Dans la UI : bind

```blueprint
Event Construct
   → Bind Event to OnHealthChanged
```

---

### Exercice 9

Créer un inventaire :

* Pickup → appelle Dispatcher “OnInventoryUpdated”
* UI → met à jour l’affichage

---

# VI — Custom Events

## Explication

Événements créés par l’utilisateur.

## Exemples

### Ouvrir une porte

```blueprint
CustomEvent OpenDoor
    → PlayTimeline
```

### Prendre des dégâts

```blueprint
CustomEvent TakeDamage(DamageAmount)
    → Health -= DamageAmount
```

---

### Exercice 10

Créer :

```
CustomEvent Explode(Force, Radius)
```

Qui :

* joue un effet
* applique RadialForce
* détruit l’acteur

---

# VII — Timers (événements retardés/bouclés)

### Explications

Ils permettent d’exécuter un event :

* après un délai
* à intervalles réguliers

---

## Exemple : piège toutes les 3s

```blueprint
BeginPlay
   → SetTimer(3s, Looping, TriggerTrap)
```

## Exemple : explosion après 5s

```blueprint
BeginPlay
   → SetTimer(5s, NonLooping, Explode)
```

---

### Exercice 11

Créer un Timer qui :

* spawn un ennemi toutes les 4s
* s’arrête après 20s

---

# EXERCICE FINAL — Projet complet : **La Salle des Événements**

Créer une salle comportant :

## 1. Porte automatique

* OnBeginOverlap → ouvrir
* OnEndOverlap → fermer

## 2. Plateforme flottante

* Sinus via Tick
* Slider UI → change la vitesse

## 3. Zone de dégâts

* Timer → dégâts périodiques
* EndOverlap → stop Timer

## 4. Lumière clignotante

* Timer → ToggleLight

## 5. UI dynamique

* Pickup → Dispatcher
* UI → mise à jour en temps réel


