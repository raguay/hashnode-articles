## Bubbletea Input Reuse

This is a continuation of my last tutorial on [Bubbletea](github.com/charmbracelet/bubbletea): [Adding a Bubbletea CLI Interface](https://blog.customct.com/adding-a-bubbletea-cli-interface). This tutorial will assume you have already read it.

## Problem With Design

After the last tutorial, there has been a lot of work fleshing out the rest of the build dialog cli. But, a core dump started happening when using views that didn’t have every input element in it. The basic design was to have many different input elements to be used in different screen layouts for the different input types. Since not all of the input types need the same information, creating an array of input types and selectively using them in each screen worked great. That is, until adding a different input type that didn’t need all the information. A core dump was the result!

The realization came that somehow the low level library was still trying to access inputs that I didn’t need for the particular screen. Even when an input wasn’t directly shown, the programing that was being used still cleared the input in the screen output. Somehow this was noticed by the underlying library and caused the errors.

The original model was:

```go
type model struct {
    savefile   string              // The file to save the structure
    orgItems   []string          // beginning list of choices
    diagItems  []string          // These are the choices for adding to a dialog
    choices    []string           // These are the current items being shown.
    cursor     int                    // which to-do list item our cursor is pointing at
    selected   int                  // which to-do items are selected
    state      int                     // What state the system is in
    textinputs []textinput.Model   // This contains the input fields for the labels
    focused    int                  // This is the currently focused input
    err        error                   // this will contain any errors from the validators
}
```

The `textinputs` variable contains all the different inputs that was used for label, input sources, and buttons. The label used all of the input type, but most of the input sources and buttons don’t. The problem comes from the methods for updating the view:

```go
// nextInput focuses the next input field
func (m *model) nextInput() {
    m.focused = (m.focused + 1) % len(m.textinputs)
}

// prevInput focuses the previous input field
func (m *model) prevInput() {
    m.focused--
    // Wrap around
    if m.focused < 0 {
        m.focused = len(m.textinputs) - 1
    }
}

func switchInLabelMode(m model, msg tea.Msg) (tea.Model, tea.Cmd) {
    var (
        cmds []tea.Cmd = make([]tea.Cmd, len(m.textinputs))
    )

    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.Type {
        case tea.KeyEnter:
            if m.focused == len(m.textinputs)-1 {
                //
                // This is the last input, save the inputs
                //
                return m, m.SaveInput
            } else {
                m.nextInput()
            }
        case tea.KeyCtrlC, tea.KeyEsc:
            return m, tea.Quit
        case tea.KeyShiftTab, tea.KeyCtrlP:
            m.prevInput()
        case tea.KeyTab, tea.KeyCtrlN:
            m.nextInput()
        }
        for i := range m.textinputs {
            m.textinputs[i].Blur()
        }
        m.textinputs[m.focused].Focus()

    // We handle errors just like any other message
    case errMsg:
        m.err = msg
        return m, nil
    }

    for i := range m.textinputs {
        m.textinputs[i], cmds[i] = m.textinputs[i].Update(msg)
    }
    return m, tea.Batch(cmds...)
}
```

The `nextInput()`, `previousInput()`, and  the `switchInLabelMode()` functions all worked with every input, not just the inputs in the current view. The crash would happen with the sending of an `Update()` message to each input in the `switchInLabelMode()` function when it wasn’t actively used in the view. It took a while to figure it out, but makes sense because I was telling the framework to update an input that wasn’t being used.

In order to use a group of inputs over and over for different views, a mechanism was needed to make sure unused inputs would not be messaged. At first, several extensive switch statements were used to make sure the right inputs were used for the right state. With more and more inputs being added, the logic was becoming too hard to manage.

## The Best Solution

The better strategy is to use an index of indexes for each view. To do this, an array of current indexes called `currentQueue` is used to hold the different indexes for each type of view. An index for each view or `queue` in the language used in the model is also added.

```go
type model struct {
	savefile     string                // The file to save the structure
	inputchoice  int                  // The input number chosen
	inputName    string            // The name for the input type being created.
	orgItems     []string            // beginning list of choices
	diagItems    []string           // These are the choices for adding to a dialog
	choices      []string            // These are the current items being shown.
	cursor       int                     // which to-do list item our cursor is pointing at
	selected     int                   // which to-do items are selected
	state        int                      // What state the system is in
	inputs       []textinput.Model      // This contains the input fields for the labels
	focused      int                   // This is the currently focused input
	currentQueue []int            // The queue of inputs to use
	labelqueue   []int               // The queue of inputs for a label
	inputqueue   []int               // The queue of inputs for a input
	buttonqueue  []int             // The queue of inputs for a button
	err          error                    // this will contain any errors from the validators
}
```

When switching to different views, the proper `labelqueue`, `inputqueue` or `buttonqueue` would be copied to the `currentQueue`. Then change all of the function for updating the inputs to use the `currentQueue` as a reference. Those function then become:

```go
// nextInput focuses the next input field
func (m *model) nextInput() {
	//
	// Increment the focused item and wrap around if
	// too large.
	//
	m.focused = (m.focused + 1) % len(m.currentQueue)
}

// prevInput focuses the previous input field
func (m *model) prevInput() {
	//
	// Decrement the focused item.
	//
	m.focused--

	//
	// If less than zero, wrap around to the highest number.
	//
	if m.focused < 0 {
		m.focused = len(m.currentQueue) - 1
	}
}

func switchInLabelMode(m model, msg tea.Msg) (tea.Model, tea.Cmd) {
	var (
		cmds []tea.Cmd = make([]tea.Cmd, len(m.inputs))
	)

	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.Type {
		case tea.KeyEnter:
			if m.focused == len(m.currentQueue)-1 {
				//
				// This is the last input, save the inputs
				//
				return m, m.SaveInput
			} else {
				m.nextInput()
			}
		case tea.KeyCtrlC, tea.KeyEsc:
			return m, tea.Quit
		case tea.KeyShiftTab, tea.KeyCtrlP:
			m.prevInput()
		case tea.KeyTab, tea.KeyCtrlN:
			m.nextInput()
		}
		for i := range m.currentQueue {
			m.inputs[m.currentQueue[i]].Blur()
		}

		//
		// Make sure the focused item is in the current queue.
		//
		if !contains(m.currentQueue, m.focused) {
			//
			// Not there, reset it.
			//
			m.focused = m.currentQueue[0]
		}

		//
		// Focus the current input.
		//
		m.inputs[m.focused].Focus()

	// We handle errors just like any other message
	case errMsg:
		m.err = msg
		return m, nil
	}

	for i := range m.currentQueue {
		m.inputs[m.currentQueue[i]], cmds[m.currentQueue[i]] = m.inputs[m.currentQueue[i]].Update(msg)
	}
	return m, tea.Batch(cmds...)
}
```

It makes for tricky indexing, but is very effective in achieving the goal with minimum programming. With this code in place and the views changed to handle a new name for each input, the code for creating the dialog is now complete. To add more input types, just increace them in the appropriate `queue` list and off it runs. No more tricky state manipulations needed!

## Example Usage

Since the main program is the cli interface also, I first make an alias for it like:

```sh
Alias bb=“/Applications/BulletinBoard.app/Contents/macOS/BulletinBoard”
```

With this alias loaded, I created the following gif to show using the new Bubletea interface for building a dialog:

![BulletinBoard Dialog Creation](https://cdn.hashnode.com/res/hashnode/image/upload/v1664446163176/8Jo1fEj1o.gif align="left”)

## Conclusion

This tutorial just shows the code that was causing the crashes and how to fix it. The complete code is in the repository for [BulletinBoard Application](https://github.com/raguay/BulletinBoard) in the `main.go` file. It is no longer a separate cli program, but is built into the main program. The next tutorial will cover processing the cli inputs using the [urfave cli](https://cli.urfave.org/) library.