# BEHAVIOR3JS

**Behavior3JS** is the original implementation and official javascript version of the Behavior3 library. It provides structures and algorithms that assist you in the task of creating intelligent agents for your game or application.

## Main features

-   Based on the work of [(Marzinotto et al., 2014)](http://www.csc.kth.se/~miccol/Michele_Colledanchise/Publications_files/2013_ICRA_mcko.pdf), in which they propose a **formal**, **consistent** and **general** definition of Behavior Trees;

-   **Optimized to control multiple agents**: you can use a single behavior tree instance to handle hundreds of agents;

-   It was **designed to load and save trees in a JSON format**, in order to use, edit and test it in multiple environments, tools and languages;

-   A **cool visual editor** which you can access online;

-   Several **composite, decorator and action nodes** available within the library. You still can define your own nodes, including composites and decorators;

-   **Completely free**, the core module and the visual editor are all published under the MIT License, which means that you can use them for your open source and commercial projects;

-   **Lightweight**!

# Usage

Install the library

```bash
npm install behavior3js
```

You can use the online visual editor to design your behavior tree.

It is very recommended that you take a look into the user guide, for both core
and editor, where you will find examples and descriptions of features in the
library:

-   https://github.com/behavior3/behavior3js/wiki

## Contents

### Core Classes

This library include the following core structures...

-   **BehaviorTree**: the structure that represents a Behavior Tree;
-   **Blackboard**: represents a "memory" in an agent and is required to to run a `BehaviorTree`;
-   **Composite**: base class for all composite nodes;
-   **Decorator**: base class for all decorator nodes;
-   **Action**: base class for all action nodes;
-   **Condition**: base class for all condition nodes;
-   **Tick**: used as container and tracking object through the tree during the tick signal;
-   **BaseNode**: the base class that provide all common node features;

### Nodes

**Composite Nodes**:

-   Sequence;
-   Priority;
-   MemSequence;
-   MemPriority;

**Decorators**:

-   Inverter;
-   Limiter
-   MaxTime;
-   Repeater;
-   RepeaterUntilFailure;
-   RepeaterUntilSuccess;

**Actions**:

-   Succeeder;
-   Failer;
-   Error;
-   Runner;
-   Wait.

## Building

In order to build the library or generate the documentation you must have **NodeJS** and **gulp**, and install the proper dependencies:

    npm install

By using

    gulp build

you get the concatenated and minified version of the library as well the documentation folder and compacted version.

Through the development, you can use

    gulp dev

in order to watch the source files and run jshint automatically.

## What I did when using this lib for Coscos Creator projects

1, define behavior tree with online editor: [b3_editor](https://www.behaviortrees.com/#/dash/home).  
port it to CocosCreator with cc.JsonAsset, connect it;

2, for Condition node, proposed to inherit B3.Condition.

2.1, define this before any CocosCreator class;

```typescript
// declare class ofr BTree inherit from b3.behaviorTree;
class isBasketVisible extends b3.Condition {}
class isUsingBall extends b3.Condition {}
```

2.2, then delcare map in CocosCreator's start function.

```typescript
    // define the start method
    start() {
        // create a new behavior tree
        this.behaviorTree = new b3.BehaviorTree();
        // create a new blackboard
        this.blackboard = new b3.Blackboard();
        // configure the behavior tree

        // TODO: Most important for b3, define tick!: <2023-05-11, Forest> //
        // this.behaviorTree.root = this.configBehaviorTree();
        let names = {
            isBasketVisible: isBasketVisible,
            isUsingBall: isUsingBall,
        };
        this.behaviorTree.load(this.kobe.json, names);

        // traverse the nodes in this.behavior.root;
        this.configTickFunction();

    }
```

3, run this.configTickFunction() to configure the tick function.

Note the line below for "expression", that's because in online editor, for each Action/Condition node, I assign a short function string for tick key;  
the function string can be found in the class that will use behavior tree;  
and use "eval" function to convert that string to a valid tick function handler.

```typescript
private configTickFunction() {
		let root = this.behaviorTree.root;
		let node_list = [];

		// add the root node to the list
		node_list.push(root);

		// traverse the tree
		while (node_list.length > 0) {
			// get the next node in the list
			let node = node_list.shift();

			// do something with the node
			// console.log(node);
			for (const prop in node.properties) {
				if (Object.prototype.hasOwnProperty.call(node.properties, prop)) {
					const expression = "this." + node.properties[prop] + ".bind(this)";
					node[prop] = eval(expression);
				}
			}

			// add the node's children to the list
			if (node.children) {
				node_list = node_list.concat(node.children);
			}
		}
	}
```

Forget about the pain to arrive here, enjoy~
