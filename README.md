import React, { useState, useEffect } from "react";
import { Button, Select, Card, CardContent } from "@/components/ui";

const API_URL = "https://api.thecatapi.com/v1/images/search";
const BREEDS_URL = "https://api.thecatapi.com/v1/breeds";
const API_KEY = "YOUR_API_KEY"; // Replace with your API key

const CatGallery = () => {
  const [cats, setCats] = useState([]);
  const [breeds, setBreeds] = useState([]);
  const [selectedBreed, setSelectedBreed] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    fetchBreeds();
  }, []);

  useEffect(() => {
    fetchCats();
  }, [selectedBreed]);

  const fetchBreeds = async () => {
    try {
      const response = await fetch(BREEDS_URL, {
        headers: { "x-api-key": API_KEY },
      });
      const data = await response.json();
      setBreeds(data); // Fetch all breeds
    } catch (error) {
      console.error("Error fetching breeds:", error);
    }
  };

  const fetchCats = async () => {
    setLoading(true);
    const headers = new Headers({
      "Content-Type": "application/json",
      "x-api-key": API_KEY,
    });

    const requestOptions = {
      method: "GET",
      headers: headers,
      redirect: "follow",
    };

    let url = `${API_URL}?size=med&mime_types=jpg&format=json&has_breeds=true&order=RANDOM&page=0&limit=10`;
    if (selectedBreed) url += `&breed_ids=${selectedBreed}`;

    try {
      const response = await fetch(url, requestOptions);
      const data = await response.json();
      setCats(data);
    } catch (error) {
      console.error("Error fetching cat images:", error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-4">
      <h1 className="text-xl font-bold mb-4">Cat Breeds</h1>
      <Select
        onChange={(e) => setSelectedBreed(e.target.value)}
        value={selectedBreed}
        className="mb-4"
      >
        <option value="">Please select</option>
        {breeds.map((breed) => (
          <option key={breed.id} value={breed.id}>{breed.name}</option>
        ))}
      </Select>
      {loading ? (
        <p>Loading...</p>
      ) : (
        <div className="grid grid-cols-2 gap-4">
          {cats.map((cat, index) => (
            <Card key={index} className="p-4">
              <CardContent>
                <img
                  src={cat.url}
                  alt="Cat"
                  className="w-full h-64 object-cover rounded-md"
                />
                {cat.breeds && cat.breeds.length > 0 && (
                  <div className="mt-4">
                    <h2 className="text-lg font-bold">
                      {cat.breeds[0].name} <span className="text-gray-500">({cat.breeds[0].origin})</span>
                    </h2>
                    <p className="text-gray-600 italic">{cat.breeds[0].id}</p>
                    <p className="mt-2">{cat.breeds[0].description}</p>
                    <a
                      href={cat.breeds[0].wikipedia_url}
                      target="_blank"
                      rel="noopener noreferrer"
                      className="text-red-500 font-bold"
                    >
                      WIKIPEDIA
                    </a>
                  </div>
                )}
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
};

export default CatGallery;
