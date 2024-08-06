# java-script-enzo
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.util.ArrayList;
import java.util.Random;

public class SnakeGame extends JPanel implements ActionListener {
    private final int GRID_SIZE = 20;
    private final int TILE_SIZE = 20;
    private final int BOARD_SIZE = GRID_SIZE * TILE_SIZE;
    private final int INIT_LENGTH = 3;

    private ArrayList<Point> snake;
    private Point food;
    private int direction; // 0=up, 1=right, 2=down, 3=left
    private boolean gameOver;
    private Timer timer;

    public SnakeGame() {
        this.setPreferredSize(new Dimension(BOARD_SIZE, BOARD_SIZE));
        this.setBackground(Color.BLACK);
        this.setFocusable(true);
        this.addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                int key = e.getKeyCode();
                if (key == KeyEvent.VK_UP && direction != 2) direction = 0;
                if (key == KeyEvent.VK_RIGHT && direction != 3) direction = 1;
                if (key == KeyEvent.VK_DOWN && direction != 0) direction = 2;
                if (key == KeyEvent.VK_LEFT && direction != 1) direction = 3;
            }
        });

        initializeGame();
        timer = new Timer(100, this);
        timer.start();
    }

    private void initializeGame() {
        snake = new ArrayList<>();
        for (int i = INIT_LENGTH - 1; i >= 0; i--) {
            snake.add(new Point(i, 0));
        }
        direction = 1;
        spawnFood();
        gameOver = false;
    }

    private void spawnFood() {
        Random rand = new Random();
        food = new Point(rand.nextInt(GRID_SIZE), rand.nextInt(GRID_SIZE));
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (gameOver) {
            timer.stop();
            repaint();
            return;
        }

        moveSnake();
        checkCollision();
        checkFood();
        repaint();
    }

    private void moveSnake() {
        Point head = snake.get(0);
        Point newHead = (Point) head.clone();

        switch (direction) {
            case 0: newHead.translate(0, -1); break;
            case 1: newHead.translate(1, 0); break;
            case 2: newHead.translate(0, 1); break;
            case 3: newHead.translate(-1, 0); break;
        }

        snake.add(0, newHead);
        snake.remove(snake.size() - 1);
    }

    private void checkCollision() {
        Point head = snake.get(0);

        if (head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE) {
            gameOver = true;
            return;
        }

        for (int i = 1; i < snake.size(); i++) {
            if (snake.get(i).equals(head)) {
                gameOver = true;
                return;
            }
        }
    }

    private void checkFood() {
        if (snake.get(0).equals(food)) {
            snake.add(new Point(-1, -1)); // Add new segment
            spawnFood();
        }
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);

        if (gameOver) {
            g.setColor(Color.RED);
            g.setFont(new Font("Arial", Font.BOLD, 40));
            g.drawString("Game Over", BOARD_SIZE / 4, BOARD_SIZE / 2);
            return;
        }

        g.setColor(Color.GREEN);
        for (Point p : snake) {
            g.fillRect(p.x * TILE_SIZE, p.y * TILE_SIZE, TILE_SIZE, TILE_SIZE);
        }

        g.setColor(Color.RED);
        g.fillRect(food.x * TILE_SIZE, food.y * TILE_SIZE, TILE_SIZE, TILE_SIZE);
    }

    public static void main(String[] args) {
        JFrame frame = new JFrame("Jogo da Serpente");
        SnakeGame gamePanel = new SnakeGame();
        frame.add(gamePanel);
        frame.pack();
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setVisible(true);
    }
}
