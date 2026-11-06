package borrowing_system1;

import javax.swing.*;
import javax.swing.border.AbstractBorder;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.RoundRectangle2D;
import java.sql.*;

public class loginv2 extends JFrame {

    // Login protection: 3 failed attempts, then a 30-second lockout.
    private int failedAttempts = 0;
    private long lockoutUntil = 0L;
    private static final int MAX_ATTEMPTS = 3;
    private static final long LOCKOUT_TIME_MS = 30_000L;

    // =========================================================
    // DATABASE CONNECTION - DO NOT CHANGE
    // =========================================================
    static final String DB_URL = "jdbc:mysql://localhost:3306/borrowing_login2";
    static final String DB_USER = "root";
    static final String DB_PASSWORD = "";

    // =========================================================
    // HIGH-TECH COLORS
    // =========================================================
    Color darkGreen = new Color(0, 255, 140);
    Color green = new Color(0, 210, 120);
    Color lightGreen = new Color(8, 18, 27);
    Color gold = new Color(0, 220, 255);
    Color darkGold = new Color(0, 150, 180);
    Color modernBackground = new Color(5, 12, 20);
    Color inputBackground = new Color(15, 29, 40);
    Color textDark = new Color(235, 250, 250);
    Color grayText = new Color(145, 165, 175);
    Color techPanel = new Color(12, 25, 35);
    Color techBorder = new Color(0, 105, 95);

    String logoPath = "logo.jpg";

    // =========================================================
    // DATABASE CONNECTION
    // =========================================================
    Connection connect() throws SQLException {
        return DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
    }

    // =========================================================
    // CONSTRUCTOR
    // =========================================================
    public loginv2() {
        setTitle("PHINMA COC Library Book Borrowing System");
        setSize(1100, 700);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);
        showLogin();
    }

    // =========================================================
    // TECH BACKGROUND
    // =========================================================
    class TechBackground extends JPanel {
        private final Timer timer;
        private float scan = 0;
        private float glow = 0;
        private int direction = 1;

        TechBackground() {
            setOpaque(true);
            timer = new Timer(45, e -> {
                scan += 1.2f;
                glow += 0.018f * direction;
                if (glow >= 1f || glow <= 0f) direction *= -1;
                repaint();
            });
            timer.start();
        }

        @Override
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            Graphics2D g2 = (Graphics2D) g.create();
            int w = getWidth();
            int h = getHeight();

            g2.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_QUALITY);
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

            GradientPaint bg = new GradientPaint(0, 0, new Color(3, 10, 17), w, h, new Color(8, 24, 31));
            g2.setPaint(bg);
            g2.fillRect(0, 0, w, h);

            // grid
            g2.setStroke(new BasicStroke(1f));
            g2.setColor(new Color(0, 100, 90, 35));
            int grid = 32;
            for (int x = 0; x < w; x += grid) g2.drawLine(x, 0, x, h);
            for (int y = 0; y < h; y += grid) g2.drawLine(0, y, w, y);

            // moving scan line
            int sy = (int) (scan % (h + 80)) - 40;
            g2.setColor(new Color(0, 255, 180, 25));
            g2.fillRect(0, sy, w, 2);

            // glow
            int gx = (int) (w * (0.25 + 0.5 * glow));
            int gy = (int) (h * 0.5);
            for (int r = 260; r > 30; r -= 20) {
                int alpha = Math.max(2, 26 - r / 14);
                g2.setColor(new Color(0, 255, 150, alpha));
                g2.fillOval(gx - r, gy - r, r * 2, r * 2);
            }

            g2.dispose();
        }

        @Override
        public void removeNotify() {
            if (timer != null) timer.stop();
            super.removeNotify();
        }
    }

    // =========================================================
    // LOGO
    // =========================================================
    JLabel createLogo(int width, int height) {
        JLabel label = new JLabel();
        ImageIcon icon = new ImageIcon(logoPath);

        if (icon.getIconWidth() > 0) {
            Image image = icon.getImage();
            Image scaled = image.getScaledInstance(width, height, Image.SCALE_SMOOTH);
            label.setIcon(new ImageIcon(scaled));
        } else {
            label.setText("PHINMA COC");
            label.setFont(new Font("Arial", Font.BOLD, 25));
            label.setForeground(darkGreen);
        }

        label.setHorizontalAlignment(SwingConstants.CENTER);
        return label;
    }

    // =========================================================
    // LOGIN SCREEN
    // =========================================================
    void showLogin() {
        TechBackground background = new TechBackground();
        background.setLayout(new GridBagLayout());
        background.setBorder(BorderFactory.createEmptyBorder(25, 25, 25, 25));

        JPanel card = new RoundedPanel(32, techPanel);
        card.setPreferredSize(new Dimension(950, 560));
        card.setLayout(new BorderLayout());
        card.setBorder(new GlowBorder(techBorder, 1, 28));

        // =====================================================
        // LEFT SIDE
        // =====================================================
        JPanel leftPanel = new JPanel();
        leftPanel.setPreferredSize(new Dimension(410, 560));
        leftPanel.setOpaque(false);
        leftPanel.setLayout(new BoxLayout(leftPanel, BoxLayout.Y_AXIS));
        leftPanel.setBorder(BorderFactory.createEmptyBorder(35, 35, 35, 35));

        leftPanel.add(Box.createVerticalGlue());

        JLabel status = new JLabel("● SYSTEM ONLINE");
        status.setFont(new Font("Arial", Font.BOLD, 11));
        status.setForeground(darkGreen);
        status.setAlignmentX(Component.CENTER_ALIGNMENT);
        leftPanel.add(status);

        leftPanel.add(Box.createVerticalStrut(15));

        JPanel logoHolder = new RoundedPanel(24, new Color(7, 19, 27));
        logoHolder.setPreferredSize(new Dimension(280, 250));
        logoHolder.setMaximumSize(new Dimension(280, 250));
        logoHolder.setLayout(new GridBagLayout());
        logoHolder.setBorder(new GlowBorder(new Color(0, 130, 115), 1, 22));
        logoHolder.add(createLogo(220, 220));
        logoHolder.setAlignmentX(Component.CENTER_ALIGNMENT);
        leftPanel.add(logoHolder);

        leftPanel.add(Box.createVerticalStrut(22));

        JLabel college = new JLabel("CAGAYAN DE ORO COLLEGE");
        college.setFont(new Font("Arial", Font.BOLD, 19));
        college.setForeground(Color.WHITE);
        college.setAlignmentX(Component.CENTER_ALIGNMENT);
        leftPanel.add(college);

        leftPanel.add(Box.createVerticalStrut(7));

        JLabel system = new JLabel("LIBRARY BORROWING SYSTEM");
        system.setFont(new Font("Arial", Font.BOLD, 13));
        system.setForeground(gold);
        system.setAlignmentX(Component.CENTER_ALIGNMENT);
        leftPanel.add(system);

        leftPanel.add(Box.createVerticalStrut(10));

        JLabel tagline = new JLabel("PHINMA EDUCATION NETWORK  //  1948");
        tagline.setFont(new Font("Arial", Font.PLAIN, 10));
        tagline.setForeground(grayText);
        tagline.setAlignmentX(Component.CENTER_ALIGNMENT);
        leftPanel.add(tagline);

        leftPanel.add(Box.createVerticalGlue());

        // =====================================================
        // RIGHT SIDE
        // =====================================================
        JPanel rightPanel = new JPanel();
        rightPanel.setOpaque(false);
        rightPanel.setLayout(new BoxLayout(rightPanel, BoxLayout.Y_AXIS));
        rightPanel.setBorder(BorderFactory.createEmptyBorder(55, 60, 45, 60));

        JLabel access = new JLabel("SECURE ACCESS // 01");
        access.setFont(new Font("Arial", Font.BOLD, 10));
        access.setForeground(darkGreen);
        access.setAlignmentX(Component.LEFT_ALIGNMENT);
        rightPanel.add(access);

        rightPanel.add(Box.createVerticalStrut(8));

        JLabel welcome = new JLabel("Welcome back!");
        welcome.setFont(new Font("Arial", Font.BOLD, 30));
        welcome.setForeground(textDark);
        welcome.setAlignmentX(Component.LEFT_ALIGNMENT);
        rightPanel.add(welcome);

        JLabel subtitle = new JLabel("Login to your library account to continue.");
        subtitle.setFont(new Font("Arial", Font.PLAIN, 13));
        subtitle.setForeground(grayText);
        subtitle.setAlignmentX(Component.LEFT_ALIGNMENT);
        rightPanel.add(Box.createVerticalStrut(6));
        rightPanel.add(subtitle);

        rightPanel.add(Box.createVerticalStrut(35));

        JLabel usernameLabel = formLabel("USERNAME");
        rightPanel.add(usernameLabel);
        rightPanel.add(Box.createVerticalStrut(7));

        JTextField username = modernTextField();
        rightPanel.add(username);

        rightPanel.add(Box.createVerticalStrut(18));

        JLabel passwordLabel = formLabel("PASSWORD");
        rightPanel.add(passwordLabel);
        rightPanel.add(Box.createVerticalStrut(7));

        JPasswordField password = modernPasswordField();
        rightPanel.add(password);

        // Show/hide password checkbox
        rightPanel.add(Box.createVerticalStrut(8));
        JCheckBox showPassword = new JCheckBox("Show Password");
        showPassword.setForeground(grayText);
        showPassword.setBackground(techPanel);
        showPassword.setFocusPainted(false);
        showPassword.setAlignmentX(Component.LEFT_ALIGNMENT);
        char defaultEchoChar = password.getEchoChar();
        showPassword.addActionListener(e -> {
            password.setEchoChar(showPassword.isSelected() ? (char) 0 : defaultEchoChar);
        });
        rightPanel.add(showPassword);

        rightPanel.add(Box.createVerticalStrut(18));

        RoundedButton loginButton = new RoundedButton("LOGIN", darkGreen);
        rightPanel.add(loginButton);

        rightPanel.add(Box.createVerticalStrut(12));

        JLabel hint = new JLabel("AUTHORIZED USERS ONLY  •  LIBRARY DATABASE");
        hint.setFont(new Font("Arial", Font.PLAIN, 9));
        hint.setForeground(new Color(95, 120, 125));
        hint.setAlignmentX(Component.LEFT_ALIGNMENT);
        rightPanel.add(hint);

        rightPanel.add(Box.createVerticalGlue());

        JLabel footer = new JLabel("PHINMA EDUCATION NETWORK  •  SECURE LIBRARY SYSTEM");
        footer.setFont(new Font("Arial", Font.PLAIN, 9));
        footer.setForeground(new Color(95, 120, 125));
        footer.setAlignmentX(Component.LEFT_ALIGNMENT);
        rightPanel.add(footer);

        // =====================================================
        // LOGIN ACTION - ORIGINAL DATABASE LOGIC
        // =====================================================
        loginButton.addActionListener(e -> {
            // Prevent login while the temporary lock is active.
            long now = System.currentTimeMillis();
            if (now < lockoutUntil) {
                long secondsLeft = (lockoutUntil - now + 999L) / 1000L;
                JOptionPane.showMessageDialog(
                        this,
                        "Too many failed attempts. Try again in " + secondsLeft + " seconds.",
                        "Login Temporarily Locked",
                        JOptionPane.WARNING_MESSAGE
                );
                return;
            }

            String user = username.getText().trim();
            String pass = new String(password.getPassword());

            if (user.isEmpty() || pass.isEmpty()) {
                JOptionPane.showMessageDialog(
                        this,
                        "Please enter username and password.",
                        "Login Required",
                        JOptionPane.WARNING_MESSAGE
                );
                return;
            }

            if (loginUser(user, pass)) {
                failedAttempts = 0;
                lockoutUntil = 0L;
                JOptionPane.showMessageDialog(
                        this,
                        "Login successful!",
                        "Success",
                        JOptionPane.INFORMATION_MESSAGE
                );
            } else {
                failedAttempts++;
                if (failedAttempts >= MAX_ATTEMPTS) {
                    lockoutUntil = System.currentTimeMillis() + LOCKOUT_TIME_MS;
                    failedAttempts = 0;
                    JOptionPane.showMessageDialog(
                            this,
                            "You entered incorrect details 3 times.\nLogin is locked for 30 seconds.",
                            "Login Temporarily Locked",
                            JOptionPane.ERROR_MESSAGE
                    );
                } else {
                    JOptionPane.showMessageDialog(
                            this,
                            "Incorrect username or password.\nAttempts remaining: "
                                    + (MAX_ATTEMPTS - failedAttempts),
                            "Login Error",
                            JOptionPane.ERROR_MESSAGE
                    );
                }
            }
        });

        card.add(leftPanel, BorderLayout.WEST);
        card.add(rightPanel, BorderLayout.CENTER);
        background.add(card);

        setContentPane(background);
        revalidate();
        repaint();
    }

    // =========================================================
    // FORM LABEL
    // =========================================================
    JLabel formLabel(String text) {
        JLabel label = new JLabel(text);
        label.setFont(new Font("Arial", Font.BOLD, 11));
        label.setForeground(new Color(185, 215, 215));
        label.setAlignmentX(Component.LEFT_ALIGNMENT);
        return label;
    }

    // =========================================================
    // TEXT FIELD
    // =========================================================
    JTextField modernTextField() {
        JTextField field = new JTextField();
        field.setMaximumSize(new Dimension(Integer.MAX_VALUE, 45));
        field.setPreferredSize(new Dimension(300, 45));
        field.setFont(new Font("Arial", Font.PLAIN, 14));
        field.setBackground(inputBackground);
        field.setForeground(textDark);
        field.setCaretColor(darkGreen);
        field.setOpaque(true);
        field.setBorder(new GlowBorder(techBorder, 1, 10));
        addFocusEffect(field);
        return field;
    }

    // =========================================================
    // PASSWORD FIELD
    // =========================================================
    JPasswordField modernPasswordField() {
        JPasswordField field = new JPasswordField();
        field.setMaximumSize(new Dimension(Integer.MAX_VALUE, 45));
        field.setPreferredSize(new Dimension(300, 45));
        field.setFont(new Font("Arial", Font.PLAIN, 14));
        field.setBackground(inputBackground);
        field.setForeground(textDark);
        field.setCaretColor(darkGreen);
        field.setOpaque(true);
        field.setBorder(new GlowBorder(techBorder, 1, 10));
        addFocusEffect(field);
        return field;
    }

    void addFocusEffect(JComponent field) {
        field.addFocusListener(new FocusAdapter() {
            @Override
            public void focusGained(FocusEvent e) {
                field.setBorder(new GlowBorder(darkGreen, 2, 10));
            }

            @Override
            public void focusLost(FocusEvent e) {
                field.setBorder(new GlowBorder(techBorder, 1, 10));
            }
        });
    }

    // =========================================================
    // LOGIN BUTTON
    // =========================================================
    void styleModernButton(JButton button, Color color) {
        button.setMaximumSize(new Dimension(Integer.MAX_VALUE, 45));
        button.setPreferredSize(new Dimension(300, 45));
        button.setBackground(color);
        button.setForeground(new Color(0, 15, 12));
        button.setFont(new Font("Arial", Font.BOLD, 12));
        button.setFocusPainted(false);
        button.setBorderPainted(false);
        button.setOpaque(true);
        button.setCursor(new Cursor(Cursor.HAND_CURSOR));
    }

    // =========================================================
    // LOGIN USER - ORIGINAL DATABASE LOGIC
    // =========================================================
    boolean loginUser(String username, String password) {
        String sql = "SELECT id, username FROM users WHERE username = ? AND password = ?";

        try (
                Connection conn = connect();
                PreparedStatement stmt = conn.prepareStatement(sql)
        ) {
            stmt.setString(1, username);
            stmt.setString(2, password);
            ResultSet rs = stmt.executeQuery();
            return rs.next();

        } catch (SQLException e) {
            JOptionPane.showMessageDialog(
                    this,
                    "Cannot connect to database.\n\n" +
                    "Error: " + e.getMessage() +
                    "\n\nCheck that XAMPP MySQL is running.",
                    "Database Error",
                    JOptionPane.ERROR_MESSAGE
            );
            return false;
        }
    }

    // =========================================================
    // ROUNDED PANEL
    // =========================================================
    class RoundedPanel extends JPanel {
        private final int radius;
        private final Color backgroundColor;

        RoundedPanel(int radius, Color backgroundColor) {
            this.radius = radius;
            this.backgroundColor = backgroundColor;
            setOpaque(false);
        }

        @Override
        protected void paintComponent(Graphics g) {
            Graphics2D g2 = (Graphics2D) g.create();
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(backgroundColor);
            g2.fillRoundRect(0, 0, getWidth(), getHeight(), radius, radius);
            g2.dispose();
            super.paintComponent(g);
        }
    }

    // =========================================================
    // ROUNDED BUTTON WITH HOVER EFFECT
    // =========================================================
    class RoundedButton extends JButton {
        private final Color normalColor;
        private boolean hover = false;

        RoundedButton(String text, Color color) {
            super(text);
            normalColor = color;
            setFont(new Font("Arial", Font.BOLD, 12));
            setForeground(new Color(0, 15, 12));
            setBackground(color);
            setFocusPainted(false);
            setBorderPainted(false);
            setContentAreaFilled(false);
            setOpaque(false);
            setMaximumSize(new Dimension(Integer.MAX_VALUE, 45));
            setPreferredSize(new Dimension(300, 45));
            setCursor(new Cursor(Cursor.HAND_CURSOR));

            addMouseListener(new MouseAdapter() {
                @Override
                public void mouseEntered(MouseEvent e) {
                    hover = true;
                    repaint();
                }

                @Override
                public void mouseEntered(MouseEvent e) {
                    hover = true;
                    repaint();
                }

                @Override
                public void mouseExited(MouseEvent e) {
                    hover = false;
                    repaint();
                }
            });
        }

        @Override
        protected void paintComponent(Graphics g) {
            Graphics2D g2 = (Graphics2D) g.create();
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

            Color fill = hover ? new Color(60, 255, 170) : normalColor;
            g2.setColor(fill);
            g2.fillRoundRect(0, 0, getWidth(), getHeight(), 14, 14);

            if (hover) {
                g2.setColor(new Color(0, 255, 180, 55));
                g2.setStroke(new BasicStroke(2f));
                g2.drawRoundRect(1, 1, getWidth() - 3, getHeight() - 3, 14, 14);
            }

            g2.dispose();
            super.paintComponent(g);
        }
    }

    // =========================================================
    // GLOW BORDER
    // =========================================================
    class GlowBorder extends AbstractBorder {
        private final Color color;
        private final int thickness;
        private final int radius;

        GlowBorder(Color color, int thickness, int radius) {
            this.color = color;
            this.thickness = thickness;
            this.radius = radius;
        }

        @Override
        public Insets getBorderInsets(Component c) {
            return new Insets(thickness + 3, thickness + 8, thickness + 3, thickness + 8);
        }

        @Override
        public Insets getBorderInsets(Component c, Insets insets) {
            insets.left = thickness + 8;
            insets.right = thickness + 8;
            insets.top = thickness + 3;
            insets.bottom = thickness + 3;
            return insets;
        }

        @Override
        public void paintBorder(Component c, Graphics g, int x, int y, int width, int height) {
            Graphics2D g2 = (Graphics2D) g.create();
            g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            g2.setColor(color);
            g2.setStroke(new BasicStroke(thickness));
            g2.drawRoundRect(x + thickness / 2, y + thickness / 2,
                    width - thickness, height - thickness, radius, radius);
            g2.dispose();
        }
    }

    // =========================================================
    // MAIN
    // =========================================================
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            loginv2 system = new loginv2();
            system.setVisible(true);
        });
    }
                
