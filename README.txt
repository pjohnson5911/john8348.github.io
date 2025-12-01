If I were to change something about this site, I'd make it so that there are no repeat images, but I didn't have time to add that in as well.

Found it cool that I can make some of the checkboxes "pre-checked" by doing like:
<label><input type = "checkbox" value = "Nature" checked> Nature</label>


Gonna write out all my comments here. I know I probably should in comments in the actual code, but I want to type a lot
to explain it more (for myself looking back) and don't wanna flood the code

Thank you to Addie, Adiva, Vadim, and Johnny for their help

---------------------------

<label>Images per category:
    <input type = "number" id = "count" value = "2" min = "1" max = "10">
  </label>

  so this section above is basically where the user chooses how many images they want generated per category.
  number takes the user input
  count is for the computer
  the value I have default at 2 just so something will generate at least
  1 is the minimum amount of images you can generate and 10 is the max


  <h3>Select Categories:</h3>
  <label><input type = "checkbox" value = "Cats"> Cats</label>
  <label><input type = "checkbox" value = "Flowers"> Flowers</label>
  <label><input type = "checkbox" value = "Technology"> Technology</label>
  <label><input type = "checkbox" value = "Dogs"> Dogs</label>
  <label><input type = "checkbox" value = "City"> City</label>

  this section is basically choosing the categories of the photos generated. 
  I can change any of them to any word and, if Wikimedia Commons has a category that matches that word in it's API, it'll grab images from there

<button id = "generateBtn">Generate Images</button>
  <div class = "status" id = "status"></div>
  <div class = "gallery" id = "gallery"></div>

  my button
  status is for errors so I can check if something goes wrong (also added in a "loading...")
  gallery is where the images go




!!! FROM THIS POINT ON I DID RECIEVE HELP FROM A VARIETY OF MY CS FRIENDS !!!



async function fetchImagesFromCategory(category, count) 
  const searchUrl =
    `https://commons.wikimedia.org/w/api.php?origin=*&format=json&action=query&generator=categorymembers&gcmtitle=Category:${encodeURIComponent(
      category
    )}&gcmtype=file&gcmlimit=50&prop=imageinfo&iiprop=url`;

sending it the category and the number of images I want
making sure the category is safe for a URL (handles spaces, etc.)
telling the API “give me members of this category”
grabbing only files (not subcategories or pages) up to 50 results
asking for image URLs

^^^ got a lot of help for this part


const response = await fetch(searchUrl);
  const data = await response.json();

  if (!data.query || !data.query.pages) return [];

  const pages = Object.values(data.query.pages);
  const images = pages.filter(p => p.imageinfo);

  if (images.length === 0) return [];

  const selected = [];
  for (let i = 0; i < count; i++) {
    const random = images[Math.floor(Math.random() * images.length)];
    selected.push(random.imageinfo[0].url);
  }

  return selected;


so we make and then wait for the network request
then convert it into a JS object

returning an empty array if what we need isn't found
convert objects into arrays
the actual images

returning an empty array if there aren't any valid images

picking a random image from the array each loop
/// IT DOES CURRENTLY PRODUCE DUPLICATES but I was too busy to fix it after I realized ///







function createCard(url, category, index) {
  const card = document.createElement("div");
  card.className = "card";

  const img = document.createElement("img");
  img.src = url;
  img.alt = category;

  const footer = document.createElement("div");
  footer.className = "footer";
  footer.textContent = `${category} — #${index + 1}`;

  card.appendChild(img);
  card.appendChild(footer);
  return card;
}

this part is just creating the image cards
something I added that I thought was interesting was alt text that includes the category name
(obviosuly it isn't GOOD alt text, but at least it's something)
just labeling the image with the category and number
adding the image and footer text to the card so each card is its own... well... card

^^^ did this section on my own



!!! This section below I got some help with again from my friends for a bit of it !!!

generateBtn.addEventListener("click", async () => {
  const count = parseInt(document.getElementById("count").value);
  const categories = [...document.querySelectorAll("input[type=checkbox]:checked")].map(c => c.value);

  if (categories.length === 0) {
    statusText.textContent = "Select at least one category. Dumbo. (Sorry)";
    return;
  }

  gallery.innerHTML = "";
  statusText.textContent = "Loading images… Don't be impatient, we're trying our best";
  generateBtn.disabled = true;

  try {
    const categoryPromises = categories.map(cat =>
      fetchImagesFromCategory(cat, count)
    );

    const allResults = await Promise.all(categoryPromises);

    allResults.forEach((imageList, idx) => {
      const category = categories[idx];
      imageList.forEach((url, i) => {
        gallery.appendChild(createCard(url, category, i));
      });
    });

    statusText.textContent = "Done! :p";
  } catch (err) {
    statusText.textContent = "Error loading images. Oopsie doopsie, we made a woopsie/";
    console.error(err);
  }

  generateBtn.disabled = false;
});

checking which chechboxes are checked
error messages and preventions (like clearing the arry each time, stopping people from pressing the button again while it's loading, etc.)
the promise section was help from a friend because I had no clue what a promise was before
building the cards for the gallery
more error message stuff



!!! The css parts are pretty self-explanatory I feel !!!