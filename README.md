# project2
import java.io.*;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

public class UrlShortener {

    private static final String BASE_URL = "http://short.ly/";
    private static final int SHORT_URL_LENGTH = 6;
    private static Map<String, String> urlMappings = new HashMap<>();
    private static final String STORAGE_FILE = "url_mappings.txt";

    public UrlShortener() {
        loadMappingsFromFile();
    }

    public String shortenUrl(String longUrl) {
        if (urlMappings.containsValue(longUrl)) {
            return getShortUrlByLongUrl(longUrl);
        }

        String shortUrl = generateShortUrl();
        while (urlMappings.containsKey(shortUrl)) {
            shortUrl = generateShortUrl();
        }

        urlMappings.put(shortUrl, longUrl);
        saveMappingsToFile();
        return BASE_URL + shortUrl;
    }

    public String expandUrl(String shortUrl) throws Exception {
        String shortCode = shortUrl.replace(BASE_URL, "");
        if (urlMappings.containsKey(shortCode)) {
            return urlMappings.get(shortCode);
        }
        throw new Exception("Invalid short URL.");
    }

    private String generateShortUrl() {
        String characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        StringBuilder shortUrl = new StringBuilder();
        Random random = new Random();

        for (int i = 0; i < SHORT_URL_LENGTH; i++) {
            shortUrl.append(characters.charAt(random.nextInt(characters.length())));
        }

        return shortUrl.toString();
    }

    private String getShortUrlByLongUrl(String longUrl) {
        for (Map.Entry<String, String> entry : urlMappings.entrySet()) {
            if (entry.getValue().equals(longUrl)) {
                return BASE_URL + entry.getKey();
            }
        }
        return null;
    }

    private void loadMappingsFromFile() {
        try (BufferedReader reader = new BufferedReader(new FileReader(STORAGE_FILE))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length == 2) {
                    urlMappings.put(parts[0], parts[1]);
                }
            }
        } catch (IOException e) {
        }
    }

    private void saveMappingsToFile() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(STORAGE_FILE))) {
            for (Map.Entry<String, String> entry : urlMappings.entrySet()) {
                writer.write(entry.getKey() + "," + entry.getValue());
                writer.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error saving URL mappings to file: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        UrlShortener urlShortener = new UrlShortener();

        String longUrl = "https://www.example.com";
        String shortUrl = urlShortener.shortenUrl(longUrl);
        System.out.println("Shortened URL: " + shortUrl);

        try {
            String expandedUrl = urlShortener.expandUrl(shortUrl);
            System.out.println("Expanded URL: " + expandedUrl);
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
