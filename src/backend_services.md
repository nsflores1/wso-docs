# Creating a Service

WSO needs a new service that presents the following API: when given a unix, it will return all data about that user.

We will be making edits to the `/services/onboarding` directory. Try to follow these steps by yourself, but if you are ever lost, look at `/services/onboarding/canonical` for an example of the fully implemented service.

**NOTE: For this entire exercise we will refer to `$UNIX`, which you should replace with your personal unix ID**

## 0. Setup

Checkout a branch from `master` called `feature/$UNIX/onboarding`: `git checkout -b feature/$UNIX/onboarding`. This branch will be your dev branch for this exercise; feel free to commit and push to this branch as much as you want.

## 1. Controller

Create the directory `/services/onboarding/$UNIX` where `$UNIX` is replaced by your actual unix. 

In that folder, create a file `controller.go`. This will be where all of your application logic goes.

We will need a structure to hold all of this business logic. This will be your `Controller` struct. Paste this in below:
```go
type Controller struct {
	services.BaseController // Inherit the base controller
	userModel *models.UserModel // DB communication to get user info
}
```
This defines our controller as inheriting properties of all WSO controllers. It also gives us database connection to get user info.

We now need a function to construct this controller. This function takes in a DB connection, server config, and a logger and outputs the controller. Within it, the function initiates the inherited controller and creates a new connection to the user DB. See below:
```go
// Construct a new onboarding controller
func NewController(db *gorm.DB, cfg *config.Config, log *zap.SugaredLogger) *Controller {
	return &Controller{
		BaseController: services.BaseController{Log: log},
		userModel:      models.NewUserModel(db, log),
	}
}
```

Before every function that is exposed by an API, we have comments that describe what that function does, so we can generate API docs. Our function should look something like this (remember to replace `$UNIX` with your unix):

```go
// GetUserByUnix godoc
// @Summary Gets a user
// @Description Gets a user from their unix. Onboarding exercise.
// @ID onboarding-$UNIX-get-user-by-unix
// @Tags onboarding
// @Accept  json
// @Produce  json
// @Param unixID path string true "Unix ID"
// @Success 200 {object} models.User
// @Failure 500 {object} lib.APIError
// @Security Bearer
// @Router /onboarding/$UNIX/{unixID} [get]
func (t *Controller) GetUserByUnix(c *gin.Context) {
	...
}
```

**Write the rest of the function.** If you don't know parts, look at the canonical example.

Hints:
 - `c.Param("unixID")` should return a the passed unix ID
 - Use `userModel.GetUserByUnixID` to get a user from a unix ID
 - Make sure to respond to errors and stop execution of the function.
 - Be sure to sanitize the user of any secret information with `sanitize.User(&user, c)`

## 2. Service Router

After we have the controller setup, we need to add it to the service's router.

A router file is made up of one Go function, `SetupRouter`, which takes in a router, a DB connection, a server config, and a logger. It then maps URLs to `Controller` functions. It is also able to require specific authorization scopes for different URLs.

In `SetupRouter`, we need to create the `Controller` from `NewController(...)`. We need to create a group that requires the user scope `auth.ScopeUsers`. Then, we need to map our URL (`/onboarding/$UNIX/
:unixID`) to our controller function (`Controller.GetUserByUnix`).

**Fill out the following function with these steps (look at `canonical` if you are stuck):**
```go
func SetupRouter(r gin.IRouter, db *gorm.DB, cfg *config.Config, log *zap.SugaredLogger) {
	...
}
```

Hints:
 - Create a group to deal with requiring the user scope: `rUser := r.Group("")`
 - To require the user scope, do `rUser.Use(auth.RequireScopes(auth.ScopeUsers))`
 - To map a URL to a function, do `rUser.GET("/:fooBar", controller.FunctionGoesHere)`, where `fooBar` can be a parameter passed by the user that is called fooBar (for this task, you might want to try doing it with `unixID`.

## 3. Unit Tests

Before we deploy anything, we should test our code. We do this with unit tests, which are used at every stage of deployment to ensure all services are working correctly.

Create a file `/services/onboarding/$UNIX/controller_test.go`. For the sake of this exercise, we will be testing on two things: does the function `GetUserByUnix` actually get a user from a unix, and does it fail when I pass in a bad unix. In a real scenario, you should be testing significantly more. You would look at what happens when this function is called without scopes or when a user data is corrupted, etc. 

You will want to have your file look something like this (this is how test files look in general):
```go
package canonical

import (
	"encoding/json"
	"net/http"
	"testing"

	"github.com/WilliamsStudentsOnline/wso-go/lib"
	utils "github.com/WilliamsStudentsOnline/wso-go/lib/test_utils"
	"github.com/WilliamsStudentsOnline/wso-go/models"
	"github.com/stretchr/testify/assert"
	"go.uber.org/zap"
)

func TestController_GetUserByUnix(t *testing.T) {
	...
}
```

**Write TestController_GetUserByUnix** Our testing suite is a bit difficult, so please feel free to look at `canonical` while writing `TestController_GetUserByUnix`. However, it is important that you get used to testing, as it is super critical.

This is how setup generally works:
```go
require := assert.New(t) // This creates a testing assertion for us to use
db := utils.SetupServiceTest(require) // This initializes our test DB

// Insert test users into db
u1 := models.User{
	Name:      "Test 1",
	UnixID:    "unix1",
	ClassYear: lib.IntToPtr(2022), // This function simply converts an integer into a pointer to an integer
} // Create a test user to put in the DB
require.NoError(db.Create(&u1).Error) // This actually puts the user into the DB and ensures it worked

// Initialize Routing
router := utils.SetupRouter(auth.ScopeUsers) // Creates a test router with the scope users. Thus, it will be authorized
utils.AddUserContexts(router, u1.ID) // Set the router to look like requests are coming from user 1. 
cfg := utils.SetupConfig() // Creates the testing server config
logger := zap.S() // Creates the testing server logger
SetupRouter(router, db, cfg, logger) // Pass all these parameters into our SetupRouter function that will set up the controller
```

Once the test is set up, we must actually test:
```go
w, err := utils.DoHTTPReq(router, http.MethodGet, "/unix1", nil) // does a GET request for unix1 on our router
require.NoError(err) // Ensure we don't fail

// Decode response
resp := utils.GetHTTPDataResp(require, w.Body.Bytes()) // Decodes the response into our API response structure (everything from the API is in this format)
require.Nil(resp.Error) // Ensure no errors from response

// Get the decoded response in respUser
respUser := models.User{} // Initialize an empty user to put the response data into
err = json.Unmarshal(resp.Data, &respUser) // Put the user data we got from the API into the user struct we just created
require.NoError(err)

// DO THE TESTS HERE:
require.Equal(u1.UnixID, respUser.UnixID) // An example test that ensures that the unixID from the user we inserted into the DB and the unixID we got from the API are the same.
// DO MORE TESTS ON NAME, ID, ETC.
```

Once you have created the tests, run it by doing `make test`. This should take awhile but if you see no errors, you know that the controller and the test work!

## 4. Root Router

Our routers are structured like a tree: many routers are called from other routers going all the way up to the root router. In order to expose our service to the outside world, we must attach our service router to some part of the tree.

Usually, we would attach a new service to the service root router at `/server/router.go`. However, for this exercise, we will use the onboarding service root router, found at `/services/onboarding/router.go`. 

Open up `/services/onboarding/router.go` and add your own service similar to how `canonical` is added.

## 5. Prepare for Pull Request

If you've followed all the steps, you should be **almost** ready to pull request this branch.

First, we want to make sure everything you have created is properly formatted, so run `make fmt`.

Second, we want to generate the new API docs, so run `make`. This also generates a new binary of the server that you can test out!

If you want to test out the server (suggested), run `./wso-backend --development`. You can now do API calls to the local server!

Once you're done testing out the server manually, be sure to run `make test` one last time before committing & pushing your code!

Finally, make a pull request to merge your branch back into master!
