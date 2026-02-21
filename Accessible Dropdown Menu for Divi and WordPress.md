# Accessible Dropdown Menu for Divi and WordPress

Dokumentacja rozwiązania poprawiającego dostępność cyfrową menu nawigacyjnego. Skrypt zmienia domyślne zachowanie menu rozwijanego (hover) na obsługę poprzez kliknięcie lub interakcję klawiaturą (click/enter), co jest zgodne z wytycznymi WCAG.

## Opis problemu

Wiele motywów WordPress (w tym Divi) opiera nawigację podmenu na zdarzeniu najechania myszą (hover). Jest to bariera dla osób:

* Korzystających wyłącznie z klawiatury.
* Korzystających z czytników ekranu (Screen Readers).
* Z niepełnosprawnościami motorycznymi.

Niniejsze rozwiązanie blokuje automatyczne rozwijanie menu po najechaniu myszą na korzyść świadomej interakcji użytkownika.

## Funkcjonalności

1. **Nawigacja klawiaturą**: Obsługa klawiszy Tab, Enter oraz Spacja.
2. **Atrybuty ARIA**: Automatyczne zarządzanie stanami aria-expanded oraz aria-haspopup.
3. **Zamykanie kontekstowe**: Możliwość zamknięcia menu klawiszem Escape lub kliknięciem poza obszar nawigacji.
4. **Responsywność**: Skrypt aktywuje się powyżej 980px, nie zakłócając działania standardowego menu mobilnego.

## Instalacja

### Krok 1: Arkusz stylów CSS

Kod należy dodać do pliku style.css motywu potomnego lub w ustawieniach Custom CSS. Wymusza on ukrycie podmenu i nadpisuje domyślne style hover motywu Divi.

```css
/* Stylizacja dostępnego menu dla Desktop (powyżej 980px) */
@media (min-width: 981px) {
    /* Ukrycie podmenu i usunięcie domyślnych przejść */
    #et-top-navigation li.menu-item-has-children > ul,
    .et_pb_menu li.menu-item-has-children > ul,
    .et_pb_fullwidth_menu li.menu-item-has-children > ul {
        display: none !important;
        visibility: hidden !important;
        opacity: 0 !important;
        transition: none !important;
    }

    /* Wyświetlanie podmenu tylko przy aktywnej klasie .dt-open */
    #et-top-navigation li.menu-item-has-children.dt-open > ul,
    .et_pb_menu li.menu-item-has-children.dt-open > ul,
    .et_pb_fullwidth_menu li.menu-item-has-children.dt-open > ul {
        display: block !important;
        visibility: visible !important;
        opacity: 1 !important;
        pointer-events: auto !important;
    }

    /* Zmiana kursora dla elementów posiadających podmenu */
    .menu-item-has-children > a {
        cursor: pointer;
    }
}

```

### Krok 2: Skrypt JavaScript (jQuery)

Skrypt należy umieścić w sekcji body lub w zewnętrznym pliku .js ładowanym po bibliotece jQuery.

```javascript
jQuery(document).ready(function($) {
    function initAccessibleMenu() {
        if ($(window).width() <= 980) return;

        var $menuItemWithChildren = $('.menu-item-has-children');
        var $link = $menuItemWithChildren.children('a');

        $link.attr('aria-haspopup', 'true').attr('aria-expanded', 'false');

        function toggleMenu(e, $el) {
            var $parent = $el.parent();
            var isOpen = $parent.hasClass('dt-open');

            if (!isOpen) {
                e.preventDefault();
                e.stopPropagation();
                
                $menuItemWithChildren.not($parent).removeClass('dt-open').children('a').attr('aria-expanded', 'false');
                $parent.addClass('dt-open');
                $el.attr('aria-expanded', 'true');
            }
        }

        $link.off('click keydown').on('click', function(e) {
            toggleMenu(e, $(this));
        });

        $link.on('keydown', function(e) {
            if (e.keyCode === 13 || e.keyCode === 32) {
                toggleMenu(e, $(this));
            }
            if (e.keyCode === 27) {
                $menuItemWithChildren.removeClass('dt-open').children('a').attr('aria-expanded', 'false');
            }
        });
    }

    initAccessibleMenu();

    $(document).on('click', function(e) {
        if (!$(e.target).closest('.menu-item-has-children').length) {
            $('.menu-item-has-children').removeClass('dt-open').children('a').attr('aria-expanded', 'false');
        }
    });

    $(window).on('resize', function() {
        if ($(window).width() <= 980) {
            $('.menu-item-has-children').removeClass('dt-open');
        } else {
            initAccessibleMenu();
        }
    });
});

```

## Obsługa nawigacji

| Klawisz | Działanie |
| --- | --- |
| Tab | Przejście do następnego elementu nawigacji |
| Enter / Spacja | Otwarcie podmenu na elemencie rodzicu |
| Escape | Zamknięcie otwartego podmenu i powrót do poziomu nadrzędnego |
| Kliknięcie myszą | Otwarcie podmenu (zamiast przejścia do strony rodzica) |

## Zgodność

Rozwiązanie zostało przetestowane z motywem Divi (Elegant Themes) oraz standardowymi modułami Menu WordPress. Wymaga biblioteki jQuery (standard w WordPress).
