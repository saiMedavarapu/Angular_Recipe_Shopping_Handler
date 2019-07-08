# Recipe Handler
This project lets the user to add recipes and list of items to the shopping list.
# Application Structure:
* Header
  * header.component.ts
* Recipes
  * Recipe-detail
    * Recipe-detail.component
  * Recipe-list
     * recipe-list component
  * Recipe-edit  
      * recipe-edit component
  * recipe-start
    * recipe-start component
  * recipeModel
  * recipes.component
  * recipe.service
* Shared
  * ingredientModel
  * Dropdown.directive
* Shoppping-list
  * ShoppinglistEdit
      * shoppinglistEdit.component
  * ShoopingList.component
  * shoppingList.service
* app.module
* app.component
* app-routing.module
  
  # NOTE:
  The documentation does not cover contents like creating components, data bindging etc. Please understand the use of @Input and @ Output decoraters for the communication between components and data binding before going through.
  
  # DropDown Directive
  This is for the manage button to manage the recipe arrow dropdown.
  ``` typescript
      @Directive({
      selector: '[appDropdown]'
    })
    export class DropdownDirective {
      @HostBinding('class.open') isOpen = false;//Dynamically attach or detach CSS class
     
      @HostListener('click') toggleOpen() { //Since I want to toggle it upon clicking it.
        this.isOpen = !this.isOpen;
      }
    }
* To be able to use it added it in the app module.
* Since this is a shared directive. This is used in recipe component and header component by using the selector in div.
  ``` HTML
    <div
      class="btn-group"
      appDropdown>//This is where we will use that directive. Please refer recipe component to see it.
      <button
        type="button"
        class="btn btn-primary dropdown-toggle">
        Manage Recipe <span class="caret"></span>
      </button>
      <ul>
        some elements in this will be dropped down.
      </ul>
   ```
# Recipe service

Servies use singleton design pattern which can be injected into components using dependency injection.
Here, Recipe service is created which will be injected into recipe component and shopping list component and used on demand.

  ``` typescript
    @Injectable()
export class RecipeService {
  recipesChanged = new Subject<Recipe[]>();
  
  private recipes: Recipe[] = [];

  constructor(private slService: ShoppingListService) {}

  setRecipes(recipes: Recipe[]) {
    this.recipes = recipes;
    this.recipesChanged.next(this.recipes.slice());
  }

  getRecipes() {
    return this.recipes.slice(); //slice returns the exact copy of the private array.
  }

  getRecipe(index: number) { //get recipe by id
    return this.recipes[index];
  }

  addIngredientsToShoppingList(ingredients: Ingredient[]) {
    this.slService.addIngredients(ingredients);
  }

  addRecipe(recipe: Recipe) {
    this.recipes.push(recipe);
    this.recipesChanged.next(this.recipes.slice());
  }

  updateRecipe(index: number, newRecipe: Recipe) {
    this.recipes[index] = newRecipe;
    this.recipesChanged.next(this.recipes.slice());
  }

  deleteRecipe(index: number) {
    this.recipes.splice(index, 1);
    this.recipesChanged.next(this.recipes.slice());
  }
}
```
## Using Serivce for the cross component communication:

* The recipe service is in recipe component. this service can be used in recipe-list component by cross component communication as below

``` typescript
this.subscription = this.recipeService.recipesChanged
      .subscribe(
        (recipes: Recipe[]) => {
          this.recipes = recipes;
        }
      );
    this.recipes = this.recipeService.getRecipes();
```
# Shopping-list Service:

* This service lets us add, delete and update ingredients.

``` typescript
export class ShoppingListService {
  ingredientsChanged = new Subject<Ingredient[]>();//This is used to emit the updated elements added to the list.
  startedEditing = new Subject<number>();
  private ingredients: Ingredient[] = [
    new Ingredient('Apples', 5),
    new Ingredient('Tomatoes', 10),
  ];

  getIngredients() {
    return this.ingredients.slice();
  }

  getIngredient(index: number) {
    return this.ingredients[index];
  }

  addIngredient(ingredient: Ingredient) {
    this.ingredients.push(ingredient);//This only returns the unadded copy 
    this.ingredientsChanged.next(this.ingredients.slice());//This emits the changed and returns them.
  }

  addIngredients(ingredients: Ingredient[]) {
    this.ingredients.push(...ingredients);
    this.ingredientsChanged.next(this.ingredients.slice());
  }

  updateIngredient(index: number, newIngredient: Ingredient) {
    this.ingredients[index] = newIngredient;
    this.ingredientsChanged.next(this.ingredients.slice());
  }

  deleteIngredient(index: number) {
    this.ingredients.splice(index, 1);
    this.ingredientsChanged.next(this.ingredients.slice());
  }
}
```
## Passing ingredients from recipes to shopping list via service:
* In the manage button of recipe component, if you want to add a recipe to the shopping list. This calls an onclick listener to a function which calls the recipe service which has access to the shopping list service then it adds to that shopping list.
``` typscript
//In the recipe edit component.
onAddToShoppingList() {
    this.recipeService.addIngredientsToShoppingList(this.recipe.ingredients);
  }
//In the Shopping list service.
addIngredients(ingredients: Ingredient[]) {
    this.ingredients.push(...ingredients);//Spread operator - allows to convert array of elements to list of elements.
    this.ingredientsChanged.next(this.ingredients.slice());
  }
```
# Adding routing using Angular router.

From Header component when clicked it should take to respective components. App-routing module is created with paths to different components. This module should be exported and javascript object of type Routes should be added to imports array in the @NgModule.

* For marking the active links. Use routerLinkActive in HTML file.
* For the child routes in the edit component.

``` typescript
const appRoutes: Routes = [
  { path: '', redirectTo: '/recipes', pathMatch: 'full' },//If nothing.
  {
    path: 'recipes',
    component: RecipesComponent,
    children: [
      { path: '', component: RecipeStartComponent },
      { path: 'new', component: RecipeEditComponent },
      {
        path: ':id',
        component: RecipeDetailComponent
      },
      {
        path: ':id/edit',
        component: RecipeEditComponent
      }
    ]
  },
  { path: 'shopping-list', component: ShoppingListComponent }//For shopping list component.
];

@NgModule({
  imports: [RouterModule.forRoot(appRoutes)], //Configuring it
  exports: [RouterModule] //Addding it to exports array.
})
export class AppRoutingModule {}
```
