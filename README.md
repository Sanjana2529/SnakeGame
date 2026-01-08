# SnakeGame

package snake_game;

public class Main {

    public static void main(String[] args) {
        new SnakeGame();
    }

}


import javax.swing.JFrame;

public class SnakeGame extends JFrame {

    SnakeGame() {

        this.add(new GamePanel());   // adds game area
        this.setTitle("Snake Game");
        this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        this.setResizable(false);
        this.pack();                 // auto size
        this.setVisible(true);       // show window
        this.setLocationRelativeTo(null); // center screen
    }
}



import javax.swing.*;     
import java.awt.*;        
import java.awt.event.*;    
import java.io.FileWriter;
import java.io.IOException;
import java.util.Random;

public class GamePanel extends JPanel implements ActionListener { 
    static final int SCREEN_WIDTH = 600;
    static final int SCREEN_HEIGHT = 600;
    static final int UNIT_SIZE = 25;
    static final int GAME_UNITS = ((SCREEN_WIDTH*SCREEN_HEIGHT)/UNIT_SIZE);
    static final int DELAY = 75;                           //speed

    // Game Variables
    final int x[] = new int[600]; // Array to hold X coordinates of the snake
    final int y[] = new int[600]; // Array to hold Y coordinates of the snake
    int bodyParts = 6; 
    int applesEaten = 0;
    int appleX, appleY;
    char direction = 'R'; 
    boolean running = false;
    Timer timer;                                                 //
    Random random;

    GamePanel() {
        random = new Random();
        this.setPreferredSize(new Dimension(SCREEN_WIDTH, SCREEN_WIDTH));
        this.setBackground(Color.black);
        this.setFocusable(true); //  react to key presses
        this.addKeyListener(new MyKeyAdapter()); 
        startGame();
    }

    public void startGame() {
        newApple();
        running = true;
        timer = new Timer(DELAY, this); 
        timer.start();
    }

    // DRAWING THE GAME
    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g); 
        draw(g);
    }

    public void draw(Graphics g) {
        if (running) {
            g.setColor(Color.red); // Apple color
            g.fillOval(appleX, appleY, UNIT_SIZE, UNIT_SIZE);

            for (int i = 0; i < bodyParts; i++) {
                if (i == 0) g.setColor(Color.green); // Head
                else g.setColor(new Color(45, 180, 0)); // Body
                g.fillRect(x[i], y[i], UNIT_SIZE, UNIT_SIZE);
            }
        } else {
            gameOver(g);
        }
    }

    // LOGIC: Generating Apple
    public void newApple() {
        appleX = random.nextInt((int) (SCREEN_WIDTH / UNIT_SIZE)) * UNIT_SIZE;
        appleY = random.nextInt((int) (SCREEN_WIDTH / UNIT_SIZE)) * UNIT_SIZE;
    }

    // LOGIC: Movement
    public void move() {
        for (int i = bodyParts; i > 0; i--) {
            x[i] = x[i - 1]; // Shift body parts: current becomes the previous one's position
            y[i] = y[i - 1];
        }
        // Move head based on direction
        if (direction == 'U') y[0] -= UNIT_SIZE;
        if (direction == 'D') y[0] += UNIT_SIZE;
        if (direction == 'L') x[0] -= UNIT_SIZE;
        if (direction == 'R') x[0] += UNIT_SIZE;
    }

    // RUBRIC
    public void saveScore() {
        try {
            FileWriter writer = new FileWriter("scores.txt", true);
            writer.write("Final Score: " + applesEaten + "\n");
            writer.close();
        } catch (IOException e) {  
            System.out.println("Could not save score.");
        }
    }

    public void checkCollisions() {
        // If head touches body
        for (int i = bodyParts; i > 0; i--) {
            if ((x[0] == x[i]) && (y[0] == y[i])) 
                running = false;
        }
        // If head touches walls
        if (x[0] < 0 || x[0] >= SCREEN_WIDTH || y[0] < 0 || y[0] >= SCREEN_WIDTH)
            running = false;

        if (!running) {
            timer.stop();
            saveScore();
        }
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (running) {
            move();
            if (x[0] == appleX && y[0] == appleY) { 
                bodyParts++;
                applesEaten++;
                newApple();
            }
            checkCollisions();
        }
        repaint(); 
    }

    public void gameOver(Graphics g) {
        g.setColor(Color.white);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("Score: " + applesEaten, 200, 300);
    }

    
    public class MyKeyAdapter extends KeyAdapter {
        @Override
        public void keyPressed(KeyEvent e) {
            switch (e.getKeyCode()) {
                case KeyEvent.VK_LEFT: if (direction != 'R') direction = 'L'; break;
                case KeyEvent.VK_RIGHT: if (direction != 'L') direction = 'R'; break;
                case KeyEvent.VK_UP: if (direction != 'D') direction = 'U'; break;
                case KeyEvent.VK_DOWN: if (direction != 'U') direction = 'D'; break;
            }
        }
    }
}
