Exmaple of an Epic:


We aim to enhance the customization capabilities of our toolkit by improving the tooling for custom components created by our customers. This will involve enabling custom components to define their orderability on a page, improving import workflow and removing complexity.

## Summary

Currently, our toolkit allows customers to create custom components but lacks the flexibility and control needed for a seamless user experience. 

## Problems:

* Customer cannot easily import a theme they started building in theme manager

* Currently we cannot easily add a custom component to our pageorderablecontainer, developers need to override the entire component which contains a lot of logic never intended to be modified by external developers.

* The storybook command introduces a lot of complexity in the toolkit that can be removed since we moved the storybook on our developer portal.

## Solutions:

* Define Orderability of Custom Components: Customers will be able to define whether their custom components can be ordered on a page or not.

* Adding custom components and overriding should be done entirely through the command line and not through copy/pasting files.  

* Overriding components should come with all its dependencies so that I have minimal issues when I update the component library

* I should easily be able to import a theme

[Appetite: 12 weeks]

## Strategic Fit

Enhancing the tooling for custom components aligns with our goal of providing a robust and user-friendly toolkit for our customers. These changes will give customers greater control over their custom components, reduce the risk of conflicts with existing components, and, ultimately, improve the overall user experience of our toolkit.


----

Example of a Story:

Right now we need to manually add the components to be overriden by the library we want to change that so the custom component is automatically replacing the components from the library.

## Acceptance Criteria

* When I deep copy a component it should be automatically loaded in the theme

* The cloned components should not be loaded anymore from sfb-theme-components

## Tech Notes:

* We should automatically generate the index file for components.

* Every time a deep clone happen the index generated is modified

* We should be clear that this file cannot be modified and is auto generated.

* Add a mention at the top of the page.


## Design Notes

No design notes for this story.