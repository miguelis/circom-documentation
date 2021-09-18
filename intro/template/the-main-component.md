# The Main component

In order to start the execution, an initial component has to be given. By default, the name of this component is “main”, and hence the component main needs to be instantiated with some template.

This is a special initial component needed to create a circuit and it defines the global input and output signals of a circuit. For this reason, compared to the other components, it has a special attribute: the list of public input signals. The syntax of the creation of the main component is:

```text
component main {public [signal_list]} = tempid(v1,...,vn);
```

where {public \[signal\_list\]} is optional. Any input signal of the template that is not included in the list is considered private. There is an example of use in the next section.

