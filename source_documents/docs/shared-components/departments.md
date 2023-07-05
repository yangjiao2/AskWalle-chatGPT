---
title: Departments
navTitle: Departments
---

#  Departments

## Description:

Departments component is used to display a nicely formatted list of common departments at Walmart, making it easier for customers to find items.

It can be viewed from **Home page** or **Search page** on the Walmart app.

<img src="images/departments1.png" width="250" />

*Home page(Departments item cell)*     

<img src="images/departments2.png" width="250" />

*Departments/Search page*

<img src="images/departments3.png" width="250" />

*Departments/Home page*

## Overview

#### DepartmentItemView

This class sets up the Item view of Departments. Item tile can contain image and title. It is of type `LDBaseView`.

###### Parameters for item:

- `image:` URL?
  - url for Image
- `shouldHideImage:` Bool
  - Boolean to hide/unhide image
- `title:` String
  - label for item title
- `categoryColor:` LDColor?
  - color for item tile


#### DepartmentsListItemCell

This class sets up cells for Department List. This class inherits from "BaseCollectionViewCell".

We also have some important functions here like `applySelectionHighlight()` which helps to change item tile color if it is selected through Boolean variables like `isSelected`, `isHighlighted`.


#### DepartmentHubspokeItemView

This class sets up Departments component on Home page and Search page.

###### Parameters:

- `imageURL:` URL?
  - url for image
- `categoryImagePlaceholderColor:` String?
  - color for image placeholder
- `categoryNameFontColor:` LDColor?
  - font color for title
- `title:` String
  - title label
- `isSmallerSize:` Bool = false
  - Boolean value to check if image is smaller or not (regular size)

###### Available ImageViewSize:
- regular = CGFloat(107)
- small = CGFloat(80)

> `isSmallerSize` helps to set us constraints when small size is selected for imageSize (default size is regular)


<img src="images/departments5.png" width="250" />

*Search page (imageSize: small)*

<img src="images/departments4.png" width="250" />

*Home page (imageSize: Regular)*
