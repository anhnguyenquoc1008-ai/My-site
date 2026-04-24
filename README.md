
package com.example.phonestore;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;

import java.util.*;

@SpringBootApplication
@RestController
public class PhoneStoreApplication {

public static void main(String[] args) {
    SpringApplication.run(PhoneStoreApplication.class, args);
}
// Fake DB
private List<Phone> phones = new ArrayList<>();
private List<Phone> cart = new ArrayList<>();
public PhoneStoreApplication() {
    phones.add(new Phone(1L, "iPhone 15", 999));
    phones.add(new Phone(2L, "Samsung S23", 800));
    phones.add(new Phone(3L, "Xiaomi 13", 500));
}
// ===== API =====
@GetMapping("/api/phones")
public List<Phone> getPhones() {
    return phones;
}
@PostMapping("/api/cart/{id}")
public List<Phone> addToCart(@PathVariable Long id) {
    phones.stream()
            .filter(p -> p.id.equals(id))
            .findFirst()
            .ifPresent(cart::add);
    return cart;
}
@GetMapping("/api/cart")
public List<Phone> getCart() {
    return cart;
}
// ===== WEBSITE =====
@GetMapping("/")
public String home() {
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>Phone Store</title>
    </head>
    <body>
        <h1>📱 Phone Store</h1>
        <div id="products"></div>
        <h2>🛒 Cart</h2>
        <ul id="cart"></ul>
        <script>
        function loadProducts() {
            fetch('/api/phones')
            .then(res => res.json())
            .then(data => {
                let html = '';
                data.forEach(p => {
                    html += `
                        <div>
                            <h3>${p.name}</h3>
                            <p>$${p.price}</p>
                            <button onclick="add(${p.id})">Add</button>
                        </div>
                    `;
                });
                document.getElementById("products").innerHTML = html;
            });
        }
        function add(id) {
            fetch('/api/cart/' + id, {method: 'POST'})
            .then(() => loadCart());
        }
        function loadCart() {
            fetch('/api/cart')
            .then(res => res.json())
            .then(data => {
                let html = '';
                data.forEach(p => {
                    html += `<li>${p.name} - $${p.price}</li>`;
                });
                document.getElementById("cart").innerHTML = html;
            });
        }
        loadProducts();
        loadCart();
        </script>
    </body>
    </html>
    """;
}
// ===== MODEL =====
static class Phone {
    public Long id;
    public String name;
    public double price;
    public Phone(Long id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}

}
