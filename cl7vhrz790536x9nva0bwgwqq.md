## Adding a Bubbletea cli Interface

In my last blog, I described the creation of the [BulletinBoard](https://github.com/raguay/BulletinBoard) application. This week I thought I would take the cli for it and add a user interface using [Bubbletea](github.com/charmbracelet/bubbletea). 

## Bubbletea

The [Bubbletea library](github.com/charmbracelet/bubbletea), and all the accompanying libraries, allow you to build a cli interface using the Model-View-Controller paradyne of programing. Programming the interface in this fashion allows you to expand the code much easier. Though it does create more boiler plate code to be written, it is worth the investment in time.

The Bubbletea library is named after a popular drink that is made where I live,  Thailand. It is small balls of tapioca starch that is died black and put in with a strong tea with milk (sometimes with sweeten condensed milk). The Thai word for it literally translates to pearls.

**Disclaimer:** This is my first cli to use the Bubbletea library and only my third program using the go language. I’m sorry if I’ve not done things improperly, but it does work! I’m open to hear of suggests to improve the program. Please join the discussion on [the discussion board](https://github.com/raguay/BulletinBoard/discussions).


## The Original CLI

This is the original cli code for sending dialogs to the userQuery program for Bulletin Board:
 
```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/aymerick/raymond"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"path"
	"path/filepath"
	"regexp"
	"strings"
)

//
// Function:          RenderDialogContents
//
// Description:       This function is used to process and render the contents of a dialog.
//
// Inputs:
//                   template      The template to use
//                   data          The data to use to render the template
//
func RenderDialogContents(template string, data map[string]string) string {
	//
	// Render the current for the first pass.
	//
	page, err := raymond.Render(template, data)
	if err != nil {
		log.Fatal(err)
	}

	//
	// Return the results.
	//
	return page
}

//
// Function:     putRequest
//
// Description:  This method will issue a put request with the data sent
//               as json in the body.
//
// Inputs:
//               url        The url to send the request
//               data       An io.Reader pointing to a json string
//
func putRequest(url string, data io.Reader) string {
	client := &http.Client{}
	req, err := http.NewRequest(http.MethodPut, url, data)
	if err != nil {
		// handle error
		log.Fatal(err)
	}

	// set the request header Content-Type for json
	req.Header.Set("Content-Type", "application/json; charset=utf-8")

	resp, err2 := client.Do(req)
	if err2 != nil {
		// handle error
		log.Fatal(err2)
	}
	body, err3 := ioutil.ReadAll(resp.Body)
	if err3 != nil {
		log.Fatal(err3)
	}
	resp.Body.Close()
	return string(body)
}

//
// Function:     fileExists
//
// Description:  This function checks if a file exists and is not a directory before we
//               try using it to prevent further errors.
//
// Inputs:       filename       A string representing the file to check.
//
func fileExists(filename string) bool {
	info, err := os.Stat(filename)
	if os.IsNotExist(err) {
		return false
	}
	return !info.IsDir()
}

//
// Function:     FilenameWithoutExtension
//
// Description:  This function trims the extension off of a file name.
//
// Inputs:
//               fn      File name to remove the extension.
//
func FilenameWithoutExtension(fn string) string {
	return strings.TrimSuffix(fn, path.Ext(fn))
}

//
// Function:     Map
//
// Description:  A utility function to return an array of strings
//               that was processed by a given function.
//
// Inputs:
//               list      Array of strings
//               f         Function to execute on each string
//
func Map(list []string, f func(string) string) []string {
	result := make([]string, len(list))
	for i, item := range list {
		result[i] = f(item)
	}
	return result
}

//
// Function:     main
//
// Description:  This is the main entry point for the program.
//
// Inputs:
//               The inputs are assigned to os.Argv. It should be a dialog
//               name and the data to use to expand it. Currently made for the
//				 macOS.
//
// #TODO: make to work for other Oses.
//
func main() {
	dialog := ""
	data := make(map[string]string, 0)
	if len(os.Args) > 1 {
		//
		// Get the command or dialog to process.
		//
		dialog = os.Args[1]

		//
		// Get the two template locations.
		//
		home := os.Getenv("HOME")
		progHome, _ := os.Executable()
		progHome = filepath.Dir(progHome)
		templates1 := filepath.Join(progHome, "../Resources/dialogs")
		templates2 := filepath.Join(home, ".config/scriptserver/dialogs")

		//
		// Process the command or the template.
		//
		switch dialog {
		case "list":
			{
				//
				// Give the user a json list of dialogs in the program
				// area and in the user directory.
				//
				var nlist []string
				file, _ := os.Open(templates1)
				dlist, _ := file.Readdirnames(0) // 0 to read all files and folders
				for _, name := range dlist {
					nlist = append(nlist, name)
				}
				file.Close()
				file, _ = os.Open(templates2)
				dlist, _ = file.Readdirnames(0)
				for _, name := range dlist {
					nlist = append(nlist, name)
				}
				file.Close()
				nlist = Map(nlist, FilenameWithoutExtension)
				pjson, err := json.Marshal(nlist)
				if err != nil {
					log.Fatal("Cannot encode to JSON ", err)
				}
				fmt.Printf("{ \"dialogs\": %s}\n", pjson)
			}
		default:
			{
				//
				// Create the rest of the command line into the data needed for the dialog template.
				//
				for i := 2; i < len(os.Args); i++ {
					data[fmt.Sprintf("data%d", i-1)] = os.Args[i]
				}

				//
				// Create an error dialog if the dialog can't be found.
				//
				var jsonStr string = "{ \"html\": \"<h1>Dialog not found.<h1>\", \"width\": 100, \"height\": 200, \"x\": 200, \"y\": 200}"

				//
				// Create the two possible file locations.
				//
				templatefile1 := filepath.Join(templates1, fmt.Sprintf("%s.json", dialog))
				templatefile2 := filepath.Join(templates2, fmt.Sprintf("%s.json", dialog))
				if fileExists(templatefile1) {
					//
					// The dialog is in the Resources directory of the application bundle
					//
					Str, _ := ioutil.ReadFile(templatefile1)
					jsonStr = string(Str)
				} else if fileExists(templatefile2) {
					//
					// The dialog is in the user's home directory area.
					//
					Str, _ := ioutil.ReadFile(templatefile2)
					jsonStr = string(Str)
				}
				if jsonStr[0] == '#' {
					//
					// This is a dialog build using a json structure.
					//
					re := regexp.MustCompile(`^#.*\r?\n`)
					jsonStr = re.ReplaceAllString(jsonStr, "")
					result := putRequest("http://localhost:9697/api/modal", strings.NewReader(jsonStr))
					fmt.Printf("%s", result[1:len(result)-1])
				} else {
					//
					// This is a raw html template that needs the data combined to finish it.
					//
					re := regexp.MustCompile(`\r?\n`)
					jsonStr = re.ReplaceAllString(jsonStr, " ")
					renderC := RenderDialogContents(jsonStr, data)
					result := putRequest("http://localhost:9697/api/dialog", strings.NewReader(renderC))
					fmt.Printf("%s", result[1:len(result)-1])
				}
			}
		}
	} else {
		//
		// Wrong information was given. Tell the user how to use the program.
		//
		// TODO: Needs better help information.
		//
		fmt.Printf("\n\nNot enough information!\nYou have to give the name of the dialog you want to show and the list of data to use in it.\nIf the only argument given is 'list', then a json list of dialogs is given.\nIf the first argument is ‘build’ with a name after it, than an interactive builder for a modal dialog will guide you through making one.")
	}
}
```

In order to compile the program, the go language compiler has to be installed. That is best done with [HomeBrew](https://brew.io). With the `brew` command, run this command line:

```sh
brew install go
```

If you build the Bulletin Board program, you should have installed go for the Wails program. Wails is built with the go language.

This program takes dialog templates, combines them with the data given on the command line, and sends it to the Bulletin Board program to display to the user. When the user inputs information and presses the button with a `submit` action, the information is then returned to the command line program. This is a great way to get user information for a script.

## Installing the Bubbletea Libraries

In order to use the Bubbletea library, we have to install it on our system and import it into our program. On the command line, run these commands:

```sh
go get github.com/charmbracelet/bubbles/textinput
go get github.com/charmbracelet/bubbletea
go get github.com/charmbracelet/lipgloss
```

These are just some of the possible Bubbletea interfaces. There are many others listed on the GitHub page for Bubbletea. In order to use them in the program, place these lines in the import section:

```go
	"github.com/charmbracelet/bubbles/textinput"
	"github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
```

Now, in the `main()` function, add this code after the  `case “list”:` block:

```go
		case "build":
			{
				//
				// We are going to build a dialog.
				//
				if len(os.Args) < 3 {
					fmt.Print("\nNot enough arguments. Please give the dialog a name!\n")
				} else {
					//
					// Initialize the buildDialog  structure. I don't have the buttons done yet, but to test
					// what I do have has to have this structure. But, every dialog needs a cancel button.
					//
					buildDialog.Buttons = make([]DialogButton, 1)
					buildDialog.Buttons[0].Name = "Cancel"
					buildDialog.Buttons[0].Id = "cancel"
					buildDialog.Buttons[0].Action = "cancel"

					//
					// create the Bubbletea interface for building the new dialog
					//
					p := tea.NewProgram(initialModel(filepath.Join(templates2, fmt.Sprintf("%s%s", os.Args[2], ".json"))))
					if err := p.Start(); err != nil {
						fmt.Printf("Alas, there's been an error: %v", err)
						os.Exit(1)
					}
				}
			}
```

First, it checks for the proper number of arguments. The `build` command needs a name for the template to build. That name is fixed with the directory for user defined dialogs and given to the `tea.NewProgram` function with the `.json` extension. If there was an error in the Bubbletea library, then an error message is printed.

I initialize one record for the `buildDialog.Buttons` because one is needed to send to the Bulletin Board program and I’m not doing that part yet. Without it, the program would not work. This tutorial is just showing the label creation. But, doesn’t every good dialog need a `cancel` button!

Now, for the rest of the Bubbletea interface. The first thing to work on is the model. Add the following code:

```go
//
// NOTE: This section is for the build cli using Bubbletea framework.
//
//
// Struct:		model
//
// description: The structure for the bubbletea interface for building a dialog.
//

//
// This is the information that we will be filling up to make the
// dialogs.
//
type DialogItem struct {
	ModelType string `json:"modaltype" bindin:"required"`
	Name      string `json:"name" bindin:"required"`
	Id        string `json:"id" bindin:"required"`
	Value     string `json:"value" bindin:"required"`
	For       string `json:"for" bindin:"required"`
}

type DialogButton struct {
	Name   string `json:"name" bindin:"required"`
	Id     string `json:"id" bindin:"required"`
	Action string `json:"action" bindin:"required"`
}

type ModalDialog struct {
	Items   []DialogItem   `json:"items" bindin:"required"`
	Buttons []DialogButton `json:"buttons" bindin:"required"`
}

var buildDialog ModalDialog // The dialog structure we need to build

type model struct {
	savefile   string            // The file to save the structure
	orgItems   []string          // beginning list of choices
	diagItems  []string          // These are the choices for adding to a dialog
	choices    []string          // These are the current items being shown.
	cursor     int               // which to-do list item our cursor is pointing at
	selected   int               // which to-do items are selected
	state      int               // What state the system is in
	textinputs []textinput.Model // This contains the input fields for the labels
	focused    int               // This is the currently focused input
	err        error             // this will contain any errors from the validators
}

type tickMsg struct{}
type errMsg error

const (
	name = iota
	id
	value
	forid
)

const (
	hotPink  = lipgloss.Color("#FF06B7")
	darkGray = lipgloss.Color("#767676")
)

var (
	inputStyle    = lipgloss.NewStyle().Foreground(hotPink)
	continueStyle = lipgloss.NewStyle().Foreground(darkGray)
)

// Validator functions to ensure valid input
func nameValidator(s string) error {
	return nil
}

func stringValidator(s string) error {
	return nil
}

func initialModel(savefile string) model {
	var inputs []textinput.Model = make([]textinput.Model, 4)
	inputs[name] = textinput.New()
	inputs[name].Placeholder = "Label Name"
	inputs[name].Focus()
	inputs[name].CharLimit = 20
	inputs[name].Width = 22
	inputs[name].Prompt = ""
	inputs[name].Validate = nameValidator

	inputs[id] = textinput.New()
	inputs[id].Placeholder = "Label Id"
	inputs[id].CharLimit = 20
	inputs[id].Width = 22
	inputs[id].Prompt = ""
	inputs[id].Validate = nameValidator

	inputs[value] = textinput.New()
	inputs[value].Placeholder = "Message"
	inputs[value].CharLimit = 20
	inputs[value].Width = 22
	inputs[value].Prompt = ""
	inputs[value].Validate = stringValidator

	inputs[forid] = textinput.New()
	inputs[forid].Placeholder = "Name of Input"
	inputs[forid].CharLimit = 20
	inputs[forid].Width = 22
	inputs[forid].Prompt = ""
	inputs[forid].Validate = nameValidator

	return model{
		// Our list of acctions
		savefile:   savefile,
		orgItems:   []string{"Add Item", "Add Button", "Save"},
		diagItems:  []string{"Add label", "Add Input", "Save"},
		choices:    []string{"Add Item", "Add Button", "Save"},
		cursor:     0,
		state:      0,
		textinputs: inputs,
		focused:    0,
		err:        nil,
	}
}

func (m model) Init() tea.Cmd {
	return textinput.Blink
}

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

type makeItemFinishedMsg struct{ m model }

func (m model) MakeItem() tea.Msg {
	return makeItemFinishedMsg{m}
}

type makeLabelFinishedMsg struct{ m model }

func (m model) MakeLabel() tea.Msg {
	return makeLabelFinishedMsg{m}
}

type makeButtonFinishedMsg struct{ m model }

func (m model) MakeButton() tea.Msg {
	return makeButtonFinishedMsg{m}
}

type makeInputFinishedMsg struct{ m model }

func (m model) MakeInput() tea.Msg {
	return makeInputFinishedMsg{m}
}

type labelInputFinishedMsg struct{ m model }

func (m model) SaveInput() tea.Msg {
	//
	// Save the input data to the build structure
	//
	switch m.state {
	case 2:
		//
		// This is the label case.
		//
		var di DialogItem
		di.ModelType = "label"
		for numti := len(m.textinputs); numti >= 0; numti-- {
			switch numti {
			case 0:
				di.Name = m.textinputs[numti].Value()
				break

			case 1:
				di.Id = m.textinputs[numti].Value()
				break

			case 2:
				di.Value = m.textinputs[numti].Value()
				break

			case 3:
				di.For = m.textinputs[numti].Value()
				break

			}
		}
		buildDialog.Items = append(buildDialog.Items, di)

	case 4:
		//
		// Creating a Input
		//
		break

	case 6:
		//
		// Creating a button
		//
		break

	default:
		break
	}

	//
	// Go back to the first state.
	//
	fmt.Print(m)
	m.state = 0
	m.cursor = 0
	m.focused = 0
	return labelInputFinishedMsg{m}
}

type saveSturctureFinishedMsg struct{ m model }

func (m model) SaveStructure() tea.Msg {
	//
	// Save the structure to a file.
	//
	file, _ := json.MarshalIndent(buildDialog, "", " ")
	header := "#\n"
	_ = ioutil.WriteFile(m.savefile, []byte(header+string(file)), 0644)
	return saveStructureFinishedMsg{m}
}
```

All of that was the model and supporting functions for the model. The model defines all the data structures we will need for creating a dialog template to use with the Bulletin Board program. It also contains the data for the input prompts for getting the information from the user. 

The model contains a `state` variable. This process of building the data structure is controlled by a state machine. This allows for putting more functionality into each function instead of having many functions. Without the state machine, it would need to setup and tear down different instances of the Bubbletea interface for each part of the data gathering process. This allows for switching input types easily.

To go from model to view to controller, Bubbletea use a messaging system controled by `tea.Msg` instances. These messages make up the side effects for the model. If you need to do file work or server calls, these functions is where you would put that type of interaction. These are also referred to as Bubbletea commands.

The `SaveInput` function saves user data from user inputs into the `buildDialog` global variable. This variable will keep the proper information until all the parts have been defined. The `buildDialog` structure is then used by the `SaveStructure` function to save it all to a JSON file.

Next are the controler functions:

```go
func switchInQueryMode(m model, msg string) (tea.Model, tea.Cmd) {
	// Cool, what was the actual key pressed?
	switch msg {

	// These keys should exit the program.
	case "ctrl+c", "q":
		return m, tea.Quit

	// The "up" and "k" keys move the cursor up
	case "up", "k":
		if m.cursor > 0 {
			m.cursor--
		}

	// The "down" and "j" keys move the cursor down
	case "down", "j":
		if m.cursor < len(m.choices)-1 {
			m.cursor++
		}

	// The "enter" key will select the action to perform.
	case "enter":
		switch m.state {
		case 0:
			if m.cursor == 1 {
				return m, m.MakeButton
			} else if m.cursor == 0 {
				return m, m.MakeItem
			} else {
				// this would save.
				return m, m.SaveStructure
			}

		case 1:
			if m.cursor == 0 {
				return m, m.MakeLabel
			} else if m.cursor == 1 {
				return m, m.MakeInput
			} else if m.cursor == 2 {
				// This would save.
				return m, m.SaveStructure
			}

		case 3:
			return m, m.MakeInput

		case 5:
			return m, m.MakeButton
		}
	}
	return m, nil
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

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg2 := msg.(type) {

	case makeItemFinishedMsg:
		m.choices = m.diagItems
		m.state = 1
		return m, nil

	case makeLabelFinishedMsg:
		m.choices = m.orgItems
		m.state = 2
		return m, nil

	case labelInputFinishedMsg:
		m.choices = m.orgItems
		m.state = 0
		return m, nil

	case makeInputFinishedMsg:
		m.choices = m.orgItems
		m.state = 0
		return m, nil

	case makeButtonFinishedMsg:
		m.state = 0
		return m, nil

	case saveSturctureFinishedMsg:
		return m, tea.Quit

	// Is it a key press?
	case tea.KeyMsg:
		switch m.state {
		case 0, 1, 3, 5:
			return switchInQueryMode(m, msg2.String())
		case 2:
			return switchInLabelMode(m, msg)
		case 6:
			return m, nil
		}
	}
	return m, nil
}
```

The `Update` function is the actual controller. It was split out into two other functions in order for it to be smaller and more readable. This helps in expanding in the future. This function controls the state switching based on the messages passed to it and the current state. It also controls what is done during keyboard interactions.

The last part is the view controllers. They are split into the view for making choice list and the view for the inputs from the user. They all return a string that the Bubbletea framework uses to update the terminal.

```go
func viewChoices(m model) string {
	// The header
	s := "\n\n\nWhat do you want to do?\n\n"

	// Iterate over our choices
	for i, choice := range m.choices {

		// Is the cursor pointing at this choice?
		cursor := " " // no cursor
		if m.cursor == i {
			cursor = ">" // cursor!
		}

		// Render the row
		s += fmt.Sprintf("%s %s\n", cursor, choice)
	}

	// The footer
	s += "\nPress j to move down. Press k to move up. Press enter to select. Press q to quit.\n\n\n\n"

	// Send the UI for rendering
	return s
}

func viewLabelInputs(m model) string {
	return fmt.Sprintf(
		` Fields for the Label

 %s
 %s
 %s
 %s
 %s  
 %s
 %s  
 %s
 %s
`,
		inputStyle.Width(10).Render("Label Name"),
		m.textinputs[name].View(),
		inputStyle.Width(2).Render("ID"),
		m.textinputs[id].View(),
		inputStyle.Width(5).Render("Value"),
		m.textinputs[value].View(),
		inputStyle.Width(6).Render("For ID"),
		m.textinputs[forid].View(),
		continueStyle.Render("Continue ->"),
	) + "\n"
}

//
// Function:    View
//
// Description: The view on a model controls how it is displayed. It returns strings
//              for displaying to the user.
//
func (m model) View() string {
	switch m.state {
	case 0, 1, 3, 5:
		return viewChoices(m)
	case 2, 4, 6:
		return viewLabelInputs(m)
	}
	return viewChoices(m)
}
```

There will be more view functions in the future for adding different elements to the dialog. As they are developed, they will be added to the states. Currently, all input states are going to the `viewLabelInputs`, but that will be updated as the program is expanded for all the other types of elements.

## Using the New CLI

With this much code, it is now possible to build a dialog with a label in it. 


![Using the Program](https://cdn.hashnode.com/res/hashnode/image/upload/v1662785920206/kwWUldqDD.gif align="left")

As can be seen, the user interface for creating the label works and creates a dialog for showing to the user.

## Conclusion

Bubbletea is a great way to make a graphical cli in the terminal for your go programs. It was fun learning and using in this project. If you want to follow my development of this project, just watch the [BulletinBoard GitHub repository](https://github.com/raguay/BulletinBoard). 

Now the work for creating the other elements in the dialog! Both the Bulletin  Board program and the queryUser program need more work. Stay tuned!