import java.io.*;
/// conectarse a internet
import java.net.HttpURLConnection;
import java.net.URL;
/// almacenar los archivos encontrados
import java.util.List;
import java.util.ArrayList;
/// manipular paginas html
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

/// clase principal
public class Descarga {

    public static void desc (String archivoURL, String guardarDireccion) {
        try {
            URL url = new URL(archivoURL);
            HttpURLConnection httpConn = (HttpURLConnection) url.openConnection();
            int responseCode = httpConn.getResponseCode(); ///codigo de respuesta del servidor (200)

            if (responseCode == HttpURLConnection.HTTP_OK) {
                String fileName = archivoURL.substring(archivoURL.lastIndexOf("/") + 1);

                InputStream inputStream = httpConn.getInputStream();
                String saveFilePath = guardarDireccion + File.separator + fileName;

                FileOutputStream outputStream = new FileOutputStream(saveFilePath);
                int bytesRead;
                /// dividir en bloques n bytes
                byte[] buffer = new byte[4096];
                while ((bytesRead = inputStream.read(buffer)) != -1) {
                    outputStream.write(buffer, 0, bytesRead);
                }
/// cerrar flujos
                outputStream.close();
                inputStream.close();
                System.out.println("Descargas: " + fileName);
            } else {
                System.out.println("No se pudo descargar el archivo.: " + responseCode);
            }
            httpConn.disconnect();
        } catch (IOException e) {
            System.out.println("Error" );
        }
    }
/// obtener enlace de cada archivo
    public static List<String> extractFileLinks(String websiteURL) {
        List<String> fileLinks = new ArrayList<>();
        /// recorre la pagina a saltos
        try {
            Document doc = Jsoup.connect(websiteURL).get();
            Elements links = doc.select("a[href]");

            for (Element link : links) {
                String href = link.attr("abs:href");
                /// tipos de archivos
                if (href.endsWith(".pdf") || href.endsWith(".doc")) {
                    fileLinks.add(href);
                }
            }
        } catch (IOException e) {
            System.out.println("Error extrayendo URLS deL sitio: " + e.getMessage());
        }
        return fileLinks;
    }

    public static void descargaMasiva (List<String> archivoURLs, String saveDir) {
        for (String fileURL : archivoURLs) {
            desc (fileURL, saveDir);
        }
    }

    public static void main(String[] args) {
        /// Insertar enlace del sitio web
        String webURL = "https://www.diputados.gob.mx/LeyesBiblio/marco.htm";
        /// Carpeta de almacenamiento de los arhivos
        String ubicacion = "C:\\Users\\dulce.lopez\\Downloads\\Archivos descargados";

        List<String> archivoURLs = extractFileLinks(webURL);

        if (!archivoURLs.isEmpty()) {
            descargaMasiva(archivoURLs, ubicacion);
        } else {
            System.out.println("No se encontraron archivos para descargar en la página.");
        }
    }
}
