# The Controlled Input Dilemma in React & Redux
*Date: January 28, 2016*

> Note: Controlled inputs with Redux never sit right with me, I wrote this article to explore the pros and cons of controlled inputs and how to handle them in React and Redux which was the most popular way in 2016.

React and Redux have transformed the way we handle forms in web applications. However, they also bring a set of challenges, especially when it comes to controlled inputs. React documentation states:

> _"An input that does not supply a value (or sets it to null) is an uncontrolled component. In a controlled input, the value of the rendered element will always reflect the value prop."_

**The Challenge with Controlled Inputs**

The handling of the `onChange` event in controlled inputs is a delicate matter. Excessive code execution during form completion can degrade the user experience, making it sluggish. This concern intensifies when using Redux due to its architecture involving actions, middlewares, and reducers, which can be performance-intensive for every keystroke.

I understand the rationale behind controlled inputs – they offer predictability and reliability™. However, the complexity and potential performance issues with forms in Redux, evidenced by the reported [performance issues with long forms in redux-form](https://github.com/rackt/redux/issues/699), make me question this approach.

**The Case for Uncontrolled Inputs**

Considering the drawbacks of controlled inputs, let’s explore keeping them uncontrolled.

1. **Handling Default Values and Validation:**
   
   React provides a `defaultValue` property for uncontrolled inputs, rendering only once and not changing with subsequent renders. This works well unless you need advanced functionalities like suggestion boxes or select-all checkboxes, which require synced states.

2. **Uncontrolled Input Validation:**

   Validation is crucial, irrespective of whether the input is controlled or uncontrolled. To address this, I developed [redux-form-validator](https://github.com/posabsolute/redux-form-validator), a small, efficient middleware for validating inputs. My focus was on fast integration, minimal interference, and scalability for future adaptations. Check out the [source code](https://github.com/posabsolute/redux-form-validator) and [demo](http://posabsolute.github.io/redux-form-validator-example/#/home).

**Controlled Inputs with Redux**

For those preferring controlled inputs, a practical approach is using `setState` on the `onChange` event and limiting Redux flow triggers to avoid performance hits, especially with long forms.

**Conclusion**

Whether to use controlled or uncontrolled inputs depends on your specific needs and concerns. If controlled inputs feel cumbersome, uncontrolled inputs with efficient validation might be your solution.

**Resources for Further Reading:**

- [Redux GitHub issue discussion on uncontrolled input](https://github.com/rackt/redux/issues/699)
- [React documentation on forms](https://facebook.github.io/react/docs/forms.html)
- [Controlled vs Uncontrolled Components in React (SitePoint Video)](http://www.sitepoint.com/video-controlled-vs-uncontrolled-components-in-react/)

What are your thoughts and experiences with controlled and uncontrolled inputs in React and Redux? Let's discuss.
