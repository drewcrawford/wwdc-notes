#css #safari 
Explore the latest advancements in CSS. Learn techniques and best practices for working with wide-gamut color, creating gorgeous typography, and writing simple and robust code. We'll also peer into the future and preview upcoming layout and typography features.

## 2:49 - Masonry layout, example 1
```css
main {
	display: grid;
  grid-template-columns: repeat(auto-fill, minmax(14rem, 1fr));
	grid-template-rows: masonry;
}
```

## 3:20 - Masonry layout, example 2
```css
main {
	display: grid;
  grid-template-columns: 1fr 2fr 3fr;
	grid-template-rows: masonry;
}
```

## 3:24 - Masonry layout, example 3
```css
main {
	display: grid;
  grid-template-columns: 10rem 1fr minmax(100px, 300px);
	grid-template-rows: masonry;
}
```

## 5:28 - Margin trim
```css
.card {
  background-color: #fcf5e7;
  padding: 2rlh;
  margin-trim: block;
}
h2, p {
  margin: 1rlh 0;
}
```

## 7:25 - Color gamut media query
```css
.card {
  background-color: #fcf5e7;
  padding: 2rlh;
  margin-trim: block;
}
h2, p {
  margin: 1rlh 0;
}
```