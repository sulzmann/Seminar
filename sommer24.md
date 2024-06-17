% Seminar Sommer 24: QuickCheck + Haskell + Rust
% Martin Sulzmann

# Inhalt

1. Seminar Übersicht

2. Teilnehmer

3. Aufgaben

4. Termine

5. Referenzen

# Seminar Übersicht

Themen: QuickCheck, Haskell, Rust

## Testen = Dynamische Programm Analyse

### Klassischer Ansatz: Unit Tests

~~~~{.cpp}
// 1. Find some input values.
string x = "Hallo";

// 2. Execute unit.
string res = reverse(x);

// 3. Check if result is correct.
assert(res == "ollaH");
~~~~~~~~~~~~~~~

Zeitintensiv, fehleranfällig, ...

### Automatisiertes Testen im Stile von QuickCheck


~~~~{.cpp}
for(int i=0; i<1000; i++) {
  string x = genString();                // 1. Automatic test data generation
  string res = reverse(x);

  assert(x.length() != res.length());    // 2. Check for invariants/properties
  assert(x == reverse(res));
}
~~~~~~~~~~~~~~~

Zum grossen Teil automatisch.

Gute Invarianten und Testdatengeneratoren sind notwendig.

## QuickCheck als Bibliothek (aka API/Framework)

Ziel: Einfache Verwendung und Erweiterbarkeit

### Quickcheck in C++ (OO Ansatz)

~~~~{.cpp}
template<typename T>
class GenOO {
public:
  virtual T generate() = 0;
};


template<typename T>
void quickCheckOO(GenOO<T>* g, bool p(T)) {

for(int i =0; i < 20; i++) {
    auto b = p(g->generate());
    if (b) {
      cout << "\n +++ OK";
    } else {
      cout << "\n *** Failed";
    }
  }

}
~~~~~~~~~~


* Object der Klasse `GenOO<T>` implementiert einen Testdaten Generator

* `quickCheck` bekommt als Parameter

    * Testdaten Generator, und

    * zu überprüfende Eigenschaft repräsentiert als Funktion `bool p(T)`


### Quickcheck in C++ (FP Ansatz)


~~~~{.cpp}
template<typename T>
class Gen {
 public:
  Gen() {};
  Gen(std::function<T()> gen_) { gen = std::function<T()>(gen_); }
  std::function<T()> gen;
  T generate() { return gen(); };
};


template<typename T>
Gen<T> arbitrary();

template<typename T>
void quickCheck(bool p(T)) {
  Gen<T> g = arbitrary<T>();

for(int i =0; i < 20; i++) {
    auto b = p(g.generate());
    if (b) {
      cout << "\n +++ OK";
    } else {
      cout << "\n *** Failed";
    }
  }

}
~~~~~~~~

Beachte:


* Generatorenobjekte sind nicht mehr notwendig

* Generatoren Instanzen werden rein durch die Typ Instanz selektiert

## Überladung (Overloading)

### C++

In der QuickCheck C++ FP Variante verwenden wir


~~~~{.cpp}
template<typename T>
Gen<T> arbitrary();
~~~~~~~~

arbitrary` Funktionen unterscheiden sich nur im Rückgabetyp.
Solch eine Form der Überladung ist eigentlich in C++ nicht möglich.
Wir können solch eine Überladungsform aber mit Hilfe von Template Instanzierung emulieren.

~~~{.cpp}
// bool
template<>
Gen<bool> arbitrary() { return Gen<bool>([] () -> bool { return (rand() % 2 == 0) ? true : false; }); }

// Build a vector of arbitrary values.
// Assumes arbitrary for T.
template<typename T>
Gen<vector<T>> vectorN(int n) {
  return Gen<vector<T>>([n] () -> vector<T>
                             {
                               vector<T> v;
                               for(int i=0; i < n; i++) {
                                  auto x = arbitrary<T>().generate();
                                  v.push_back(x);
                                }
			       return v;
			     });
}
~~~~~~~~~~

### Haskell

Die erste QuickCheck Version wurde in Haskell entwickelt.

Haskell hat einen mächtigeren Überladungsmechanismus (via Typklassen).

~~~~{.haskell}
class Arbitrary a where
   arbitrary :: IO a


quickCheck prop = go 100
   where go 0 = putStrLn "+++ Ok"
         go n = do x <- arbitrary                        -- Generate test input data x
                   if prop x                             -- Check if x satisfies property prop
                     then go (n-1)
                     else do putStrLn "*** Failed: "
                             print x
~~~~~~~~~~~


Haskell behandelt Seiteneffekt (Zufallszahlen) via Monaden (siehe `IO`).

Typen sind optional. Haskell hat "volle" Typinferenz.

Haskell erlaubt die Definition neuer Datentypen und (überladener) Funktionen welche diese Datentypen verwenden.


~~~~~{.haskell}
data Either a b = Left a | Right b

instance (Arbitrary a, Arbitrary b) => Arbitrary (Either a b) where
   arbitrary = do b <- randomIO
                  if b
                    then do l <- arbitrary
                            return $ Left l
                    else do r <- arbitrary
                            return $ Right r
~~~~~~~~~~~


Haskell ist *esoterisch*.

<b>Haskell's slogan: Avoid success at all costs!</b>

This means. Don't make compromises when evolving the language. Be bold. Try out many new things.


## Rust (versus C++ und Haskell)


Rust ist effektiv Haskell aber ohne automatischen Speicherverwaltung.

Rust verwendet die C++ "move semantics". Etwaige Speicherfehler werden aber statisch (zur Compilezeit) abgefangen.


Hier ist eine Skizze von "QuickCheck" in Rust.

~~~~{.rs}
trait Arbitrary {
  fn arbitrary() -> Self;
}

fn quick_check<T:Arbitrary>(p : fn(T)->bool) {
    for _i in 1..100 {
           let x : T = Arbitrary::arbitrary();
          if p(x) {
              println!("OK");
          } else {
              println!("FAIL");
          }
    }
}


enum Either<T,S> {
    Left(T),
    Right(S)
}

impl<T:Arbitrary,S:Arbitrary> Arbitrary for Either<T,S> {
    fn arbitrary() -> Either<T,S> {
        let x : T = Arbitrary::arbitrary();
        let y : S = Arbitrary::arbitrary();
        let b : bool = 1 > 5; // some random number, couldn't figure out "rnd" in Rust
        if b {
            return Either::Left(x);
        } else {
            return Either::Right(y);
        }
    }

}
~~~~~~~~~~~

# Teilnehmer

1. [QuickCheck in C++](https://github.com/MGZeroes/quickcheck-cpp) (Daniel Cindric)
2. [QuickCheck in anderen Programmiersprachen](https://github.com/bale1017/seminar) (Leonard Bausenwein)
3. [QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs]( https://github.com/s-kuhn/seminar-haskell-qc) (Sascha Kuhn)

4. [Ownership Types in Rust](https://github.com/PhilKrau/OwnershipTypesRust) (Phil Krause)
5. [Ownership types fuer Einsteiger in Rust](https://github.com/michael-gleike/rustviz) (Michael Gleike)
6. [C++ templates/overloading versus Rust traits](https://github.com/deco42/seminar_cpp_templates-overloading_vs_rust_traits) (Jeremy Zolk)
7. Traits und Enums in Rust (Matthias Handtmann)




* Am besten legen sie ein github repo an.

* Readme.md als Dokumentation

* Informeller Vortragsstil, siehe "Seminar Übersicht".


# Aufgaben

## Termin 13.06

### C++ Templates

Standard Beispiele, z.B. vector und typische Fehlermeldungen

### Rust ownership

Komplexere Fehlermeldungen. Beispiele.

Als Quelle mag folgendes Papier [A Grounded Conceptual Model for Ownership Types in Rust](https://dl.acm.org/doi/pdf/10.1145/3622841).
Dieses Papier scheint im direkten Bezug zu [aquascope](https://github.com/cognitive-engineering-lab/aquascope) zu stehen.

### QuickCheck

Generierung von Strings nach Vorgabe (siehe unten).

1. Korrektheit

2. Effizienz

## Abgeschlossen (13.05)

### Aufgabe QuickCheck Anwendung

Bezieht sich auf die Themen QuickCheck-Haskell, QuickCheck-Java und QuickCheck-C++ (rapidcheck)

1. Definiere eine `arbitrary` Instanz für Strings wobei folgendes gelten soll

* Der String enhält nur Kleinbuchstaben und Leerzeichen

* Der String darf maximal die Länge 40 haben

* Es dürfen maximal 10 Leerzeichen vorkommen, wobei hintereinander nie mehr als 5 Leerzeichen vorkommen dürfen

2. Definiere eine `arbitrary` Instanz für `data List a = Nil | Cons a (List a)` (bzw den entsprechenden Datentyp in C++ und Java) wobei folgendes gelten soll

* die Liste darf maximal 10 Elemente besitzen

### Aufgabe C++ Template Instanziierung

1. Betrachte typische Beispiele

2. Welche Einschränkungen gibt es?

### Aufgabe Rust enums und traits

1. Betrachte Beispiele

2. Ziehe Vergleich zu Haskell's data types und type classes

### Aufgabe Rust ownership types

1. Diskutiere Rust Typfehlermeldungen (wegen Speicherfehler)

2. Gibt es Tools welche dem User helfen diese Fehler zu beseitigen?


## Abgeschlossen (02.05)

*Rust ownership.*

* Ziehe einen Vergleich zur C++ "move semantics" mittels einfacher Beispiele (z.B. verwende Beispiele bekannt aus der SoftwareProjekt Vorlesung)

* Betrachte RustViz


# Termine

Jeweils Donnerstag ab 14:00. Raum siehe Termin.

Nach jedem Termin/Meeting finden sich Aktionspunkte und Fragen welche als Denkanstoss gedacht sind.
Sie sollen sich mit diesen Aktionspunkten und Fragen beschäftigen.
Eine Besprechung erfolgt im folgenden Meeting.


## 25.04 (E201)

- Einführung kurzer Vortrag zu "QuickCheck, Haskell und Rust" etwa 20min
- Sammlung offener Punkte
- Diskussion und Besprechung weiterer Schritte

## 02.05 (E301)

- Jeder Teilnehmer stellt Ergebnisse und offene Punkte vor (etwa 5min)
- Sammlung offener Punkte
- Diskussion und Besprechung weiterer Schritte

## 16.05 (E201)

- Jeder Teilnehmer stellt Ergebnisse und offene Punkte vor (etwa 5min)
- Sammlung offener Punkte
- Diskussion und Besprechung weiterer Schritte

## 13.06 (E206)

- Abschlussvortrag, Hr. Cindric und Hr. Handtmann jeweils etwa 10-15 Minuten

- Restlichen Teilenehmer gehen auf die offenen Aufgaben (kurz und knapp) ein

## Zoom

Falls Teilnahme in Präsenz nicht möglich hier der zoom link

https://h-ka-de.zoom-x.de/j/4837536496?pwd=dnlrTmVhWXlYOTFNMEhnYVNtRTJwZz09

Unter Umständen kommt noch ein 5ter Termin hinzu.
Die Abschlussvorträge finden zusammen mit der Seminargruppe Fuchß und Sinz.
Insgesamt sind wir etwa 14 Teilnehmer, deshalb reicht der 13.06 wahrscheinlich nicht.


# Referenzen

## QuickCheck

Die Idee von [QuickCheck](https://en.wikipedia.org/wiki/QuickCheck)
basiert auf einer [Haskell Bibliothek](https://hackage.haskell.org/package/QuickCheck).
QuickCheck Implementierungen gibt es aber jetzt in den verschiedensten Programmiersprachen.

Die Idee von QuickCheck wurde in folgenden Vorlesung kurz vorgestellt.

[Softwareprojekt (2tes Semester)](https://sulzmann.github.io/SoftwareProjekt/lec-cpp-advanced-quick-check.html)

[Programming Paradigms (Master)](https://sulzmann.github.io/ProgrammingParadigms/lec-haskell-testing.html)


## C++, Rust, Haskell

[Softwareprojekt](https://github.com/sulzmann/SoftwareProjekt)

[Modell-basiert](https://github.com/sulzmann/ModelBasedSW)

[Programming Paradigms](https://github.com/sulzmann/ProgrammingParadigms)
