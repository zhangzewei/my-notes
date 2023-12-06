# Use tailwind in atomic design methodology 
> author: markzzw  &nbsp;&nbsp;&nbsp;  date: 2023-12-6

Tailwind CSS has gradually become the preferred style processing solution in front-end development. Its simple integration and flexible configurability make it gradually occupy a higher position in front-end development. However, with the use of Tailwind CSS, many front-end developers have also experienced discomfort. Too many classnames cause the code to look untidy, and in the case of complex styles, writing Tailwind CSS is very complicated, which is not conducive to maintenance, so can the above problems be solved by following the theory of atomic components when using Tailwind CSS?

## Atomic Component
Before discussing how to combine atomized components and Tailwind CSS, let's take a look at the atomized component methodology, which divides pages into the following five levels.
1. Atoms
    
    For the definition of the atom as the smallest non-divisible component in the design system, then we should split the web page should be the corresponding html native tag with the corresponding style, that is, input/h1/button and other components, such components can not be split again, and can be used independently.
2. Molecules

    A molecule is defined as a component of the design system composed of atomic components, such as a search box with a search button, avatar, card, etc
3. Organisms

    An organization is defined as a component composed of atoms and molecules in the design system. Such components generally have certain business capabilities, such as tab components, datepicker components, and list components
4. Templates

    The template is defined as a high-reuse interface without data composed of atoms, molecules, and organizations in the design system, such as the table page of the background management system, which has high reuse.
5. Pages

    The definition of the page is that after the template is filled with data, the real page is displayed, also known as a high-fidelity page.

Each level is pieced together from the previous levels to form the final page. In this methodology, the main thing is to find atomic components. The following figure helps us to label the atomic components well (native Html elements).

## Atomic Component fit with Tailwind CSS