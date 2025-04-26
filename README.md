# LAB2_EDA
import java.util.*;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

class Voto {
    private int id;
    private int votante_id;
    private int candidato_id;
    private String timestamp;
    public Voto(int id, int votante_id, int candidato_id) {
        this.id = id;
        this.votante_id = votante_id;
        this.candidato_id = candidato_id;
        this.timestamp = LocalTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
    }
    public int get_id() { return id; }
    public int get_votante_id() { return votante_id; }
    public int get_candidato_id() { return candidato_id; }
    public String get_timestamp() { return timestamp; }
    public String to_string() {
        return "Voto #" + id + " de votante " + votante_id + " para candidato " + candidato_id + " a las " + timestamp;
    }
}

class Candidato {
    private int id;
    private String nombre;
    private String partido;
    private Queue<Voto> votos_recibidos;
    public Candidato(int id, String nombre, String partido) {
        this.id = id;
        this.nombre = nombre;
        this.partido = partido;
        this.votos_recibidos = new LinkedList<>();
    }
    public int get_id() { return id; }
    public String get_nombre() { return nombre; }
    public String get_partido() { return partido; }
    public Queue<Voto> get_votos_recibidos() { return votos_recibidos; }
    public void agregar_voto(Voto v) {
        votos_recibidos.add(v);
    }
    public int contar_votos() {
        return votos_recibidos.size();
    }
    public String to_string() {
        return nombre + " (" + partido + ") - Votos: " + contar_votos();
    }
}

class Votante {
    private int id;
    private String nombre;
    private boolean ya_voto;
    public Votante(int id, String nombre) {
        this.id = id;
        this.nombre = nombre;
        this.ya_voto = false;
    }
    public int get_id() { return id; }
    public String get_nombre() { return nombre; }
    public boolean get_ya_voto() { return ya_voto; }
    public void marcar_como_votado() {
        ya_voto = true;
    }
    public String to_string() {
        return nombre + " (ID: " + id + ") - " + (ya_voto ? "Ya votó" : "Puede votar");
    }
}

class UrnaElectoral {
    private LinkedList<Candidato> lista_candidatos;
    private LinkedList<Votante> lista_votantes;
    private Stack<Voto> historial_votos;
    private Queue<Voto> votos_reportados;
    private int id_counter;
    public UrnaElectoral() {
        this.lista_candidatos = new LinkedList<>();
        this.lista_votantes = new LinkedList<>();
        this.historial_votos = new Stack<>();
        this.votos_reportados = new LinkedList<>();
        this.id_counter = 1;
    }
    public void agregar_candidato(Candidato candidato) {
        lista_candidatos.add(candidato);
    }
    public void agregar_votante(Votante votante) {
        lista_votantes.add(votante);
    }
    public Candidato buscar_candidato(int id) {
        for (Candidato c : lista_candidatos) {
            if (c.get_id() == id) {
                return c;
            }
        }
        return null;
    }
    public Votante buscar_votante(int id) {
        for (Votante v : lista_votantes) {
            if (v.get_id() == id) {
                return v;
            }
        }
        return null;
    }
    public boolean verificar_votante(Votante votante) {
        return votante.get_ya_voto();
    }
    public boolean registrar_voto(Votante votante, int candidato_id) {
        if (verificar_votante(votante)) {
            System.out.println("ERROR: Este votante ya votó.");
            return false;
        }
        Candidato candidato = buscar_candidato(candidato_id);
        if (candidato == null) {
            System.out.println("ERROR: Candidato no existe.");
            return false;
        }
        Voto nuevo_voto = new Voto(id_counter++, votante.get_id(), candidato_id);
        candidato.agregar_voto(nuevo_voto);
        historial_votos.push(nuevo_voto);
        votante.marcar_como_votado();
        System.out.println("Voto registrado exitosamente!");
        System.out.println(nuevo_voto.to_string());
        return true;
    }
    public boolean reportar_voto(Candidato candidato, int id_voto) {
        Queue<Voto> votos_candidato = candidato.get_votos_recibidos();
        for (Voto voto : votos_candidato) {
            if (voto.get_id() == id_voto) {
                if (votos_reportados.contains(voto)) {
                    System.out.println("ADVERTENCIA: Este voto ya fue reportado anteriormente.");
                    return false;
                }
                votos_reportados.add(voto);
                votos_candidato.remove(voto);
                System.out.println("Voto #" + id_voto + " reportado exitosamente.");
                return true;
            }
        }
        System.out.println("ERROR: Voto no encontrado para este candidato.");
        return false;
    }
    public void mostrar_resultados() {
        System.out.println("RESULTADOS ACTUALES:");
        for (Candidato c : lista_candidatos) {
            System.out.println(c.to_string());
        }
    }
    public void mostrar_resultados_detallados() {
        System.out.println("RESULTADOS DETALLADOS:");
        for (Candidato c : lista_candidatos) {
            System.out.println(c.to_string());
            System.out.println("  Votos recibidos:");
            for (Voto v : c.get_votos_recibidos()) {
                System.out.println("    " + v.to_string());
            }
        }
        System.out.println("\nVOTOS REPORTADOS/ANULADOS:");
        if (votos_reportados.isEmpty()) {
            System.out.println("  No hay votos reportados");
        } else {
            for (Voto v : votos_reportados) {
                System.out.println("  " + v.to_string());
            }
        }
    }
    public void mostrar_instrucciones() {
        System.out.println("INSTRUCCIONES:");
        System.out.println("1. Registrar votantes primero");
        System.out.println("2. Registrar candidatos después");
        System.out.println("3. Cada votante puede votar solo una vez");
        System.out.println("4. Seleccione el ID del candidato al votar");
    }
    private void emitir_voto(Scanner scanner) {
        System.out.println("LISTA DE CANDIDATOS:");
        if (lista_candidatos.isEmpty()) {
            System.out.println("No hay candidatos registrados aún.");
            return;
        }
        for (Candidato c : lista_candidatos) {
            System.out.println("ID: " + c.get_id() + " - " + c.get_nombre() + " (" + c.get_partido() + ")");
        }
        System.out.print("Ingrese su ID de votante: ");
        int votante_id = scanner.nextInt();
        Votante votante = buscar_votante(votante_id);
        if (votante == null) {
            System.out.println("ERROR: Votante no registrado.");
            return;
        }
        if (votante.get_ya_voto()) {
            System.out.println("ERROR: Este votante ya ejerció su voto.");
            return;
        }
        System.out.print("Ingrese ID del candidato elegido: ");
        int candidato_id = scanner.nextInt();
        registrar_voto(votante, candidato_id);
    }
    private void registrar_nuevo_votante(Scanner scanner) {
        System.out.print("Ingrese ID del nuevo votante: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        if (buscar_votante(id) != null) {
            System.out.println("ERROR: ID de votante ya existe.");
            return;
        }
        System.out.print("Ingrese nombre del votante: ");
        String nombre = scanner.nextLine();
        agregar_votante(new Votante(id, nombre));
        System.out.println("Votante registrado exitosamente!");
    }
    private void registrar_nuevo_candidato(Scanner scanner) {
        System.out.print("Ingrese ID del nuevo candidato: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        if (buscar_candidato(id) != null) {
            System.out.println("ERROR: ID de candidato ya existe.");
            return;
        }
        System.out.print("Ingrese nombre del candidato: ");
        String nombre = scanner.nextLine();
        System.out.print("Ingrese partido del candidato: ");
        String partido = scanner.nextLine();
        agregar_candidato(new Candidato(id, nombre, partido));
        System.out.println("Candidato registrado exitosamente!");
    }
    public void mostrar_menu_principal() {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("MENU PRINCIPAL");
            System.out.println("1. Registrar votante");
            System.out.println("2. Registrar candidato");
            System.out.println("3. Votar");
            System.out.println("4. Ver resultados");
            System.out.println("5. Ver resultados detallados");
            System.out.println("6. Ver instrucciones");
            System.out.println("7. Reportar voto");
            System.out.println("8. Salir");
            System.out.print("Seleccione opción: ");
            int opcion = scanner.nextInt();
            switch (opcion) {
                case 1:
                    registrar_nuevo_votante(scanner);
                    break;
                case 2:
                    registrar_nuevo_candidato(scanner);
                    break;
                case 3:
                    emitir_voto(scanner);
                    break;
                case 4:
                    mostrar_resultados();
                    break;
                case 5:
                    mostrar_resultados_detallados();
                    break;
                case 6:
                    mostrar_instrucciones();
                    break;
                case 7:
                    System.out.print("Ingrese ID del candidato: ");
                    int candidato_id = scanner.nextInt();
                    System.out.print("Ingrese ID del voto a reportar: ");
                    int voto_id = scanner.nextInt();
                    Candidato c = buscar_candidato(candidato_id);
                    if (c != null) {
                        reportar_voto(c, voto_id);
                    } else {
                        System.out.println("Candidato no encontrado");
                    }
                    break;
                case 8:
                    System.out.println("Saliendo del sistema...");
                    scanner.close();
                    return;
                default:
                    System.out.println("Opción inválida, intente nuevamente.");
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        UrnaElectoral urna = new UrnaElectoral();
        urna.agregar_votante(new Votante(1, "Juan Perez"));
        urna.agregar_votante(new Votante(2, "Maria Gonzalez"));
        urna.agregar_candidato(new Candidato(1, "Ana Lopez", "Partido Verde"));
        urna.agregar_candidato(new Candidato(2, "Carlos Ruiz", "Partido Azul"));
        System.out.println("SISTEMA DE VOTACIONES - BIENVENIDO");
        urna.mostrar_menu_principal();
    }
}
