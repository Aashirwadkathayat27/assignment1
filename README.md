import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Repository;
import org.springframework.web.bind.annotation.*;
import java.util.ArrayList;
import java.util.List;


public class ServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServerApplication.class, args);
    }
}

class Server {
    private int id;
    private String name;
    private String ipAddress;

    public Server(int id, String name, String ipAddress) {
        this.id = id;
        this.name = name;
        this.ipAddress = ipAddress;
    }

    
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getIpAddress() {
        return ipAddress;
    }

    public void setIpAddress(String ipAddress) {
        this.ipAddress = ipAddress;
    }
}

interface ServerRepository {
    List<Server> findAll();
    Server findById(int id);
    Server findByName(String name);
    void save(Server server);
    void deleteById(int id);
}


class InMemoryServerRepository implements ServerRepository {
    private List<Server> servers = new ArrayList<>();

    public InMemoryServerRepository() {
        servers.add(new Server(1, "Server 1", "192.168.0.1"));
        servers.add(new Server(2, "Server 2", "192.168.0.2"));
    }

    
    public List<Server> findAll() {
        return servers;
    }

    @Override
    public Server findById(int id) {
        return servers.stream()
                .filter(server -> server.getId() == id)
                .findFirst()
                .orElse(null);
    }

    
    public Server findByName(String name) {
        return servers.stream()
                .filter(server -> server.getName().equalsIgnoreCase(name))
                .findFirst()
                .orElse(null);
    }

    
    public void save(Server server) {
        servers.add(server);
    }

    
    public void deleteById(int id) {
        servers.removeIf(server -> server.getId() == id);
    }
}


@RequestMapping("/servers")
class ServerController {
    private final ServerRepository serverRepository;

    public ServerController(ServerRepository serverRepository) {
        this.serverRepository = serverRepository;
    }

    
    public List<Server> getAllServers() {
        return serverRepository.findAll();
    }

    
    public ResponseEntity<Server> getServerById(@PathVariable int id) {
        Server server = serverRepository.findById(id);
        if (server == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(server);
    }

    @GetMapping("/name/{name}")
    public ResponseEntity<Server> getServerByName(@PathVariable String name) {
        Server server = serverRepository.findByName(name);
        if (server == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(server);
    }

    
    public ResponseEntity<Void> createServer(@RequestBody Server server) {
        serverRepository.save(server);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }

    
    public ResponseEntity<Void> updateServer(@PathVariable int id, @RequestBody Server server) {
        Server existingServer = serverRepository.findById(id);
        if (existingServer == null) {
            return ResponseEntity.notFound().build();
        }
    }
}   
