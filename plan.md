# PLAN

## Le CPU, le langage machine et les langages de haut niveau

 * le process d'un CPU : fetch/decode/execute depuis des nombres en mémoire
 * ces nombres sont des identifiants d'instructions, des opcodes
 * les opcodes forment donc un langage : le langage machine, le seul compris par le CPU
 * 3 familles d'instructions : mémoire, opérations, contrôle de flux
 * on peut tout faire en langage machine
 * on le recopiait même à la main
 * pour se simplifier la vie, du texte à la place des opcodes et un convertisseur : le compilateur
 * la mise en place de concepts et paradigmes de plus haut niveau : encore des compilateurs

-----

## Le fonctionnement d'un moteur Javascript

 * l'archi des machines très éloignées du Web et JS ?
 * oh que non !

### Les compilateurs internes, l'optimisation et la déoptimisation

 * le traitement du JS par un navigateur/moteur JS : un compilo dans le moteur !
 * compilation rapide
 * profiling et optimisation via un compilo plus lent
 * de-optimisation si nécessaire !
 * usage d'un interpréteur de bytecode

### Le problème du Javascript pour la performance
 
 * la compilation : le même traitement que sur nos machines de dev ! (webpack, uglify, etc.)
 * un langage super pour les devs, super pénible pour un compilo qui veut générer du code machine rapide
 * variable faiblement typées
 * objets à formes variables

-----

## Les alternatives du passé

### Les plugins
 * ActiveX, Flash
 * NaCl, PNaCl

### La tentative asm.js

 * un sous-ensemble de JS pour aider les compilateurs (référence à "asm.js")
 * à permis d'atteindre 2x native (attention à cette notion)
 * mais un code lourd et toujours aussi lent à compiler vers le langage machine

-----

## Web Assembly, la solution ?

### Quelques idées reçues sur WA

  * WA est un concurrent/remplaçant de JS : non !
  * WA offre des performances natives : oui et non. Il règle plusieurs autres problèmes : poids, boot...
  * WA nous donne accès au coeur de la machine puisque c'est de l'assembleur : non !

### Définition de WA

 * un format binaire d'instructions pour une machine virtuelle basée sur le concept de pile et une cible portable pour des langages de haut niveau. WUT ????!!!!
 * slide avec code WA brut
 * process parallèle à la compilation JS (slide WASM opcode to machine opcode)
 * Web Assembly est en prison : aucun accès à l'extérieur, on peut juste jouer avec de la mémoire en sandbox
 * C'est une micro-machine virtuelle sans rien à l'intérieur : on doit faire soi-même (malloc, free, etc.)
 * Pour les accès extérieurs on a les imports/exports : à partir de là on peut tout faire
 * C'est le host qui est responsable de la sécurité. Dans le cas du Web : WebGL, WebAudio, WebSockets, accès au DOM, etc.
 * Le navigateur est il un OS ?

### Let's play ! 

## Pour la culture, low-level : Web Assembly Text format (WAT)
 
 * S-Expression, WAT
 * les outils : WABT (wabbit) à intégrer dans des tools chains. wat2wasm, wasm2wat, wat-desugar, wasm-interp
 * les outils : Binaryen
 * les basiques : import, export, functions, piles (exemple avec add42To)
 * wat2wasm et instanciation avec WebAssembly.instantiateStreaming
 * live coding ?

## C/C++, emscripten
 
 * le WAT est globalement inutilisable mais on peut envoyer du C/C++/Rust
 * rappel : tout doit être fait à la mano (malloc, printf, etc.)
 * la libc dans le navigateur fournie par la glue emscripten


-------------------------------------------------------------------------------------------------

 Bordel : 

  * tout est à faire : la mémoire est linéaire, le contrôle est total
  * WA est bloquant
  * donné une liste des outils et leurs rôles :  WABT, Binaryen et Emscripten

A approfondir :

emscripten  build une libc (comment ? où ça ? -> vider cache). 
Les fonctions utilisées dans le code C (genre malloc) sont alors embarquées dans le WASM.
-> A vérifier (mais comment ?)
Avec mon essai jpeg j'ai un Module.malloc. D'où il sort ???

Les réponses : 
 * Principe : emscripten fait plein de choses pour nous. En premier lieu il compile du C vers du WASM. 
 * Comme la notion de string de JS est très éloignée de celle de C (char* vs UTF8 au format interne du moteur JS), si l'on veut passer une string en argument on est obligé d'allouer de la mémoire au sens WASM (un i32 qui fait office de ptr), de convertir la string JS et de la copier dans cette mémoire.
 * Les fonctions pour allouer et convertir sont fournies par emscripten. Mais pour les obtenir il faut les inclure dans le build. Emscripten nous donne donc une glue
 * La glue est générée automatiquement avec ce que voit emscripten. Donc si je compile un fichier C, il verra mes fonctions C et les incluera dans la glue. 
 * Si ces fonctions ne sont pas visibles (par exemple dans allocate ou intArrayFromString dans index.html) alors il faut les importer à la main au moment du build : emcc simple-c.c -o simple-c.js -s WASM=1 -s NO_EXIT_RUNTIME=1 -s 'EXTRA_EXPORTED_RUNTIME_METHODS=["allocate", "intArrayFromString", "Pointer_stringify"]'

A noter, vouloir loader soi-même un WASM généré par emscripten est une mauvaise idée : la glue s'occupe de générer l'objet d'import vers le WASM et tout un tas de trucs.

Ce qu'il faut retenir : à partir du moment où on passe en C on est déjà sur un langage de haut niveau avec une gestion implicite de plein de choses : appels de fonctions et pile, allocation, etc.

-----------------------------------------------------

Binaryen : 
emscripten repose sur Binaryen (d'ailleurs emcc -s WASM=1 est un synonyme de emcc -S BINARYEN=1)