Saving State
=======================================
So, you’ve built a marvellous app; configured your layout and your user has arranged things just the way they like. Great! 

The next time the user opens the app they'll want to find everything just the way they left it. Or (if you’re feeling fancy) let them choose from a number of saved layouts.

GoldenLayout offers a powerful persistence mechanism for layout state. The state of the components within the layout can be converted into a serialisable object that can be saved to a database remotely or even in the browser's local storage.

Here’s how it works:

### Creating the layout
For this tutorial we'll use the layout created in [getting-started](getting-started.html).

	var config = {
		content: [{
			type: 'row',
			content:[{
				type: 'component',
				componentName: 'testComponent',
				componentState: { label: 'A' }
			},{
				type: 'column',
				content:[{
					type: 'component',
					componentName: 'testComponent',
					componentState: { label: 'B' }
				},{
					type: 'component',
					componentName: 'testComponent',
					componentState: { label: 'C' }
				}]
			}]
		}]
	};

var myLayout = new GoldenLayout( config );

### Listening for state changes

Your layout instance and all the items within it emit events. These bubble up, just like DOM events. The event we're interested in at the moment is called `stateChanged`. It is emitted whenever something happens that modifies the saveable layout state. Listening to it works like this:

	myLayout.on( 'stateChanged', function(){
		//now save the state
	});

The actual state object is created by calling `myLayout.toConfig();`. For our example we'll store it in the browser's localStorage.

	myLayout.on( 'stateChanged', function(){
		var state = JSON.stringify( myLayout.toConfig() );
		localStorage.setItem( 'savedState', state );
	});

<div class="info">You might wonder why you have to call `myLayout.toConfig()` explicitly rather than just getting the new state as a parameter of the `stateChanged` callback. This is because serialisinging entire layouts can be expensive - and `stateChanged` will fire a lot. It might therefor be a good idea to 'debounce' your state save calls or offer a save-button, depending on your performance requirements.</div>

### Creating Layouts from saved states

So now the next time the user opens the app there's a choice. Either he has used it before and the app's state is stored
in localStorage or he's using it for the first time and we provide a default config.

	var myLayout,
		savedState = localStorage.getItem( 'savedState' );

	if( savedState !== null ) {
		myLayout = new GoldenLayout( JSON.parse( savedState ) );
	} else {
		myLayout = new GoldenLayout( config );
	}

### Saving Component States

So far we've only saved the layout's state, but what about the components within it - the ones you've built? Well, remember the `componentState: { label: 'C' }` entry that you've configured? This is just the initial state, the component itself can update it by calling `container.extendState( state );` or `container.setState( state );`.

This stores it and emits a `stateChanged` event that bubbles up to the layout manager.

Let's update our `testComponent` to show a text input with a persistent value:

	myLayout.registerComponent( 'testComponent', function( container, state ){

		// Create the input
        var input = $( '<input type="text" />' );
        
        // Set the initial / saved state
        if( state.label ) {
            input.val( state.label );
        }
		
		// Store state updates
        input.on( 'change', function(){
            container.extendState({
                label: input.val()
            });
        });

		// Append it to the DOM
        container.getElement().append( input );
    });

### And initialise the layout
	myLayout.init();

### The result
<p data-height="268" data-theme-id="7376" data-slug-hash="7c599be2a33fb57a47dfb43a53df2437" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/wolframhempel/pen/7c599be2a33fb57a47dfb43a53df2437/'>Saving State</a> by Wolfram Hempel (<a href='http://codepen.io/wolframhempel'>@wolframhempel</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
