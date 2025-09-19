#### RecipeFilter.jsx
Before:

```javascript
const handleFilterChange = (type, value) => {
  filters[type] = value;
  onFilterChange(filters);
};
```

After:

```javascript
const handleFilterChange = (filterType, value) => {
  const newFilters = { ...filters };
  if (['allergens','country','mainIngredient'].includes(filterType)) {
    newFilters[filterType] = filters[filterType].includes(value)
      ? filters[filterType].filter(i => i !== value)
      : [...filters[filterType], value];
  } else newFilters[filterType] = value;
  onFilterChange(newFilters);
};
```

Improvement:
- Supports multi-select filters.
- Avoids mutating state directly.
- Handles allergen, country, and ingredient selections properly.

#### RecipeDetail.jsx

Before:

```javascript
useEffect(() => {
  fetch(`http://localhost:5001/api/recipes/${id}`)
    .then(res => res.json())
    .then(data => setRecipe(data));
}, [id]);
```

After:

```javascript
useEffect(() => {
  const fetchRecipeDetail = async () => {
    try {
      const response = await fetch(`http://localhost:5001/api/recipes/${id}`);
      if (!response.ok) throw new Error('Recipe not found');
      const data = await response.json();
      setRecipe(data.data);
      setIsBookmarked(data.data.isBookmarked || false);
    } catch (err) { setError(err.message); }
  };
  fetchRecipeDetail();
}, [id]);
```

Improvement:
- Added error handling and loading state.
- Sets isBookmarked for UI.
- Uses async/await for cleaner code.

##### RecipeDetail.jsx – Bookmark Button

Before:

```javascript
<button onClick={() => setIsBookmarked(!isBookmarked)}>Bookmark</button>
```

After:

```javascript
<button onClick={handleBookmarkToggle} className="absolute top-4 right-4 p-2 bg-white rounded-full shadow-md">
  {isBookmarked ? <FilledBookmarkIcon /> : <EmptyBookmarkIcon />}
</button>
```

Improvement:
- Connected UI to backend toggle.
- Conditional rendering for filled/empty bookmark.
- Added hover and shadow styling.

#### RecipeFilter.jsx – Selected Filters Tags

Before:

```javascript
{filters.allergens.map(a => <span>{a}</span>)}
```

After:

```javascript
{filters.allergens.map(a => (
  <div key={a} className={`inline-flex items-center px-4 py-2 rounded-full text-sm font-medium ${getAllergenColor(a)}`}>
    ⚠️ {a}
    <button onClick={(e) => { e.stopPropagation(); handleRemoveFilter('allergens', a); }} className="ml-2">❌</button>
  </div>
))}
```

Improvement:
- Added remove button for each tag.
- Styled tags for clarity.
- Multi-select filters can now be removed individually.

#### RecipeDetail.jsx – Star Rating

Before:

```javascript
<span>{recipe.rating}</span>
```

After:

```javascript
<div className="flex items-center">
  {renderStars(recipe.rating)}
  <span className="text-gray-600 font-medium ml-2">{recipe.rating}</span>
</div>
```
Improvement:
- Added star icons to visually represent rating.
- Half stars handled for fractional ratings.
- Improves readability and UX.


#### RecipeFilter.jsx – Display Active Filter Tags

Before:

```javascript
<span>{filters.country}</span>
```

After:

```javascript
{filters.country.map(country => (
  <div key={country} className="inline-flex items-center px-4 py-2 rounded-full bg-blue-100 text-blue-800">
    <span>🌍 {country}</span>
    <button onClick={() => handleRemoveFilter('country', country)}>❌</button>
  </div>
))}
```

Improvement:
- Shows active filters as removable tags.
- Supports multi-select removal.
- Clear visual feedback for applied filters.