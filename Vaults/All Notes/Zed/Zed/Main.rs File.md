_____
**Created**: 26-03-2025 06:28 pm
**Status**: In Progress
**Tags**: #Rust #Zed #Text_Editor
**References**: Zed Github Repo
______

### Functions
- `flies_not_created_on_launch`
	- This function declares a message that says *Zed failed to launch.*, gets the error details as a string.
	- Creates a new application than prompts the user with the error details, and exits if cancelled I think. 🤔

#### Main
- Starts with `let args = Args::parse()`.
-  I'm an idiot these calls are made to preserve the binaries since they would be optimized away if the zed binary is not using it. To circumvent this they just call empty init functions.
- Then there is a `menu::init()` which does nothing.
- Now there is a `zed_actions::init()` which I think similarly does nothing.
- There are checks for whether stdout is a pseudo terminal or not. This also drives the logger creation.
- **Check Why The Fuck is a weak pointer being used and when reopening trying to upgrade it to an `ARC`.**