import javax.swing.*;
import javax.swing.border.Border;
import javax.swing.border.CompoundBorder;
import javax.swing.border.EmptyBorder;
import javax.swing.border.LineBorder;
import java.awt.*;
import java.awt.event.*;
import java.io.IOException; // For audio loading
import java.net.URL;      // For audio loading
import java.util.Enumeration;
import javax.sound.sampled.*; // For audio playback

class Scoreboard {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new ScoreboardGUI());
    }
}

class ScoreboardGUI extends JFrame {
    // === Re-added Field Declarations ===
    // Team specific components
    private JTextField team1NameField, team2NameField;
    private JLabel score1Label, score2Label;
    private JLabel timeouts1DisplayLabel, timeouts2DisplayLabel; // For team panels
    private JLabel[] team1FoulMarkerLabels, team2FoulMarkerLabels; // For foul markers

    private int score1 = 0, score2 = 0;
    private int fouls1 = 0, fouls2 = 0;
    private int timeouts1 = 3, timeouts2 = 3;
    private final int INITIAL_TIMEOUTS = 3;
    private final int MAX_FOULS_DISPLAY = 5;
    private final String FOUL_ACTIVE_CHAR = "●";
    private final String FOUL_INACTIVE_CHAR = "○";

    // Main Timer components
    private JLabel mainTimerLabel;
    private Timer mainTimer;
    private int mainTimerSeconds = 0;
    private boolean mainTimerRunning = false;
    private boolean mainTimerCountsUp = false; // Default to countdown with presets
    private JButton startPauseMainTimerButton;
    private ButtonGroup timerPresetGroup;
    private JRadioButton radio5min, radio10min, radio12min;

    // Shot Clock components
    private JLabel shotClockLabel;
    private Timer shotClockTimer;
    private int shotClockSeconds = 24;
    private final int DEFAULT_SHOT_CLOCK_DURATION = 24;
    private boolean shotClockRunning = false;
    private JButton startShotClockButton;
    // === End of Re-added Field Declarations ===

    // Audio Clip for Horn Sound
    private Clip hornSoundClip;

    // === Re-added Styling Constants and Fonts ===
    // New Styling Constants
    private final Color NEW_APP_BG = new Color(44, 62, 80); // Dark Slate Blue
    private final Color NEW_DISPLAY_BOX_BG = new Color(52, 73, 94); // Wet Asphalt
    private final Color NEW_TEXT_COLOR_PRIMARY = new Color(236, 240, 241); // Clouds (off-white)
    private final Color NEW_TEAM1_ACCENT = new Color(52, 152, 219); // Peter River Blue
    private final Color NEW_TEAM2_ACCENT = new Color(231, 76, 60); // Alizarin Crimson
    private final Color NEW_BUTTON_BG = new Color(93, 109, 126); // Grayish Blue
    private final Color NEW_BUTTON_FG = Color.WHITE;
    private final Color NEW_MAIN_TIMER_FG = new Color(46, 204, 113); // Emerald Green
    private final Color NEW_SHOT_CLOCK_FG = new Color(241, 196, 15); // Sun Flower Yellow
    private final Color NEW_FOUL_MARKER_INACTIVE_FG = new Color(127, 140, 141); // Asbestos Gray
    private final Color NEW_CONTROL_PANEL_BG = new Color(40,50,70); // Slightly darker for controls

    // Fonts
    private final Font FONT_SANS_LARGE_TITLE = new Font("Segoe UI", Font.BOLD, 28);
    private final Font FONT_SANS_MEDIUM_LABEL = new Font("Segoe UI", Font.PLAIN, 18);
    private final Font FONT_SANS_BUTTON = new Font("Segoe UI", Font.BOLD, 14);
    private final Font FONT_SANS_RADIO = new Font("Segoe UI", Font.PLAIN, 13);
    private final Font FONT_DIGITAL_HUGE = new Font("Consolas", Font.BOLD, 90);  // Score
    private final Font FONT_DIGITAL_LARGE = new Font("Consolas", Font.BOLD, 72); // Main Timer
    private final Font FONT_DIGITAL_MEDIUM = new Font("Consolas", Font.BOLD, 48);// Shot Clock
    private final Font FONT_FOUL_MARKER = new Font("Arial", Font.BOLD, 28); // For ● ○ symbols
    // === End of Re-added Styling Constants and Fonts ===


    public ScoreboardGUI() {
        setTitle("Scoreboard");
        setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE); // Handle closing manually to release audio
        addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                if (hornSoundClip != null) {
                    hornSoundClip.close(); // Release audio resources
                }
                System.exit(0); // Exit the application
            }
        });

        setMinimumSize(new Dimension(1100, 700));
        setSize(1200, 750);
        getContentPane().setBackground(NEW_APP_BG);
        setLayout(new BorderLayout(10, 10));

        loadHornSound(); // Load the sound effect

        JPanel topTimerPanel = createTopTimerPanel();
        add(topTimerPanel, BorderLayout.NORTH);

        JPanel teamsDisplayPanel = new JPanel(new GridLayout(1, 2, 15, 0));
        teamsDisplayPanel.setOpaque(false);
        teamsDisplayPanel.setBorder(new EmptyBorder(5, 10, 5, 10));
        teamsDisplayPanel.add(createSingleTeamDisplayPanel(1, "HOME", NEW_TEAM1_ACCENT));
        teamsDisplayPanel.add(createSingleTeamDisplayPanel(2, "GUEST", NEW_TEAM2_ACCENT));
        add(teamsDisplayPanel, BorderLayout.CENTER);

        JPanel mainControlsPanel = createMainControlsPanel();
        add(mainControlsPanel, BorderLayout.SOUTH);

        initializeUIData();
        radio5min.setSelected(true);
        setMainTimerDurationAction(5 * 60);
        updateMainTimerLabel();

        setVisible(true);
    }

    private void loadHornSound() {
        try {
            URL soundUrl = getClass().getResource("/horn.wav");
            if (soundUrl == null) {
                System.err.println("Horn sound file (horn.wav) not found in classpath. Buzzer will use system beep.");
                return;
            }
            AudioInputStream audioIn = AudioSystem.getAudioInputStream(soundUrl);
            hornSoundClip = AudioSystem.getClip();
            hornSoundClip.open(audioIn);
        } catch (UnsupportedAudioFileException | IOException | LineUnavailableException e) {
            System.err.println("Error loading horn sound: " + e.getMessage());
            e.printStackTrace();
            hornSoundClip = null;
        }
    }

    private JPanel createTopTimerPanel() {
        JPanel panel = new JPanel(new FlowLayout(FlowLayout.CENTER, 0, 5));
        panel.setOpaque(false);
        panel.setBorder(new EmptyBorder(10,0,0,0));

        mainTimerLabel = new JLabel("05:00", SwingConstants.CENTER);
        mainTimerLabel.setFont(FONT_DIGITAL_LARGE);
        mainTimerLabel.setForeground(NEW_MAIN_TIMER_FG);
        mainTimerLabel.setOpaque(true);
        mainTimerLabel.setBackground(NEW_DISPLAY_BOX_BG);
        mainTimerLabel.setBorder(new CompoundBorder(
                new LineBorder(NEW_MAIN_TIMER_FG.darker(), 2),
                new EmptyBorder(5, 20, 5, 20)
        ));
        panel.add(mainTimerLabel);
        return panel;
    }

    private JPanel createSingleTeamDisplayPanel(int teamId, String defaultName, Color accentColor) {
        JPanel panel = new JPanel(new GridBagLayout());
        panel.setOpaque(false);
        panel.setBorder(new LineBorder(accentColor, 2, true));
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(8, 10, 8, 10);
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.weightx = 1.0;

        JTextField teamNameField = new JTextField(defaultName);
        teamNameField.setFont(FONT_SANS_LARGE_TITLE);
        teamNameField.setForeground(NEW_TEXT_COLOR_PRIMARY);
        teamNameField.setBackground(NEW_DISPLAY_BOX_BG);
        teamNameField.setHorizontalAlignment(JTextField.CENTER);
        teamNameField.setCaretColor(accentColor);
        teamNameField.setBorder(new CompoundBorder(
                new LineBorder(accentColor.darker()),
                new EmptyBorder(8, 8, 8, 8)
        ));
        gbc.gridx = 0; gbc.gridy = 0; gbc.gridwidth = 2;
        panel.add(teamNameField, gbc);

        JLabel scoreLabel = new JLabel("00", SwingConstants.CENTER);
        scoreLabel.setFont(FONT_DIGITAL_HUGE);
        scoreLabel.setForeground(NEW_TEXT_COLOR_PRIMARY);
        scoreLabel.setOpaque(true);
        scoreLabel.setBackground(NEW_DISPLAY_BOX_BG);
        scoreLabel.setBorder(new EmptyBorder(10,5,10,5));
        gbc.gridy = 1; gbc.ipady = 20;
        panel.add(scoreLabel, gbc);
        gbc.ipady = 0;

        JLabel foulsTextLabel = new JLabel("FOULS:", SwingConstants.LEFT);
        foulsTextLabel.setFont(FONT_SANS_MEDIUM_LABEL);
        foulsTextLabel.setForeground(NEW_TEXT_COLOR_PRIMARY);
        gbc.gridy = 2; gbc.gridwidth = 1; gbc.anchor = GridBagConstraints.LINE_START; gbc.fill = GridBagConstraints.NONE;
        panel.add(foulsTextLabel, gbc);

        JPanel foulMarkersDisplayPanel = new JPanel(new FlowLayout(FlowLayout.LEFT, 3, 0));
        foulMarkersDisplayPanel.setOpaque(false);
        JLabel[] markerLabels = new JLabel[MAX_FOULS_DISPLAY];
        for (int i = 0; i < MAX_FOULS_DISPLAY; i++) {
            markerLabels[i] = new JLabel(FOUL_INACTIVE_CHAR, SwingConstants.CENTER);
            markerLabels[i].setFont(FONT_FOUL_MARKER);
            markerLabels[i].setForeground(NEW_FOUL_MARKER_INACTIVE_FG);
            foulMarkersDisplayPanel.add(markerLabels[i]);
        }
        gbc.gridx = 1; gbc.anchor = GridBagConstraints.LINE_END;
        panel.add(foulMarkersDisplayPanel, gbc);

        JLabel timeoutsTextLabel = new JLabel("TIMEOUTS:", SwingConstants.LEFT);
        timeoutsTextLabel.setFont(FONT_SANS_MEDIUM_LABEL);
        timeoutsTextLabel.setForeground(NEW_TEXT_COLOR_PRIMARY);
        gbc.gridx = 0; gbc.gridy = 3; gbc.anchor = GridBagConstraints.LINE_START;
        panel.add(timeoutsTextLabel, gbc);

        JLabel timeoutDisplay = new JLabel(String.valueOf(INITIAL_TIMEOUTS));
        timeoutDisplay.setFont(FONT_SANS_MEDIUM_LABEL);
        timeoutDisplay.setForeground(accentColor);
        gbc.gridx = 1; gbc.anchor = GridBagConstraints.LINE_END;
        panel.add(timeoutDisplay, gbc);

        if (teamId == 1) {
            this.team1NameField = teamNameField;
            this.score1Label = scoreLabel;
            this.team1FoulMarkerLabels = markerLabels;
            this.timeouts1DisplayLabel = timeoutDisplay;
        } else {
            this.team2NameField = teamNameField;
            this.score2Label = scoreLabel;
            this.team2FoulMarkerLabels = markerLabels;
            this.timeouts2DisplayLabel = timeoutDisplay;
        }

        gbc.gridy = 4; gbc.weighty = 1.0;
        panel.add(Box.createVerticalGlue(), gbc);

        return panel;
    }

    private JPanel createMainControlsPanel() {
        JPanel panel = new JPanel(new GridBagLayout());
        panel.setBackground(NEW_CONTROL_PANEL_BG);
        panel.setBorder(new EmptyBorder(10, 10, 10, 10));
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.insets = new Insets(4, 4, 4, 4);

        JPanel team1CtrlPanel = createSingleTeamControls(1, "Team 1", NEW_TEAM1_ACCENT);
        gbc.gridx = 0; gbc.gridy = 0; gbc.gridheight = 2; gbc.weightx = 0.3;
        gbc.fill = GridBagConstraints.BOTH;
        panel.add(team1CtrlPanel, gbc);
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.gridheight = 1;

        JPanel clockCtrlPanel = new JPanel(new GridBagLayout());
        clockCtrlPanel.setOpaque(false);
        GridBagConstraints cgbc = new GridBagConstraints();
        cgbc.fill = GridBagConstraints.HORIZONTAL;
        cgbc.insets = new Insets(3, 0, 3, 0);
        cgbc.gridwidth = GridBagConstraints.REMAINDER;
        cgbc.weightx = 1.0;

        JPanel presetPanel = new JPanel(new FlowLayout(FlowLayout.CENTER, 5, 0));
        presetPanel.setOpaque(false);
        timerPresetGroup = new ButtonGroup();
        radio5min = createStyledDarkRadioButton("5 Min", e -> setMainTimerDurationAction(5 * 60));
        radio10min = createStyledDarkRadioButton("10 Min", e -> setMainTimerDurationAction(10 * 60));
        radio12min = createStyledDarkRadioButton("12 Min", e -> setMainTimerDurationAction(12 * 60));
        timerPresetGroup.add(radio5min); timerPresetGroup.add(radio10min); timerPresetGroup.add(radio12min);
        presetPanel.add(radio5min); presetPanel.add(radio10min); presetPanel.add(radio12min);
        cgbc.gridy = 0; clockCtrlPanel.add(presetPanel, cgbc);

        JPanel mainTimerBtns = new JPanel(new GridLayout(1,2,5,0));
        mainTimerBtns.setOpaque(false);
        startPauseMainTimerButton = createStyledDarkButton("START", NEW_MAIN_TIMER_FG, NEW_BUTTON_BG, e -> toggleMainTimer());
        mainTimerBtns.add(startPauseMainTimerButton);
        mainTimerBtns.add(createStyledDarkButton("RESET", NEW_TEAM2_ACCENT, NEW_BUTTON_BG, e -> resetMainTimer()));
        cgbc.gridy = 1; clockCtrlPanel.add(mainTimerBtns, cgbc);

        JLabel scLabel = new JLabel("Shot Clock", SwingConstants.CENTER);
        scLabel.setFont(FONT_SANS_MEDIUM_LABEL.deriveFont(14f));
        scLabel.setForeground(NEW_TEXT_COLOR_PRIMARY.darker());
        cgbc.gridy = 2; cgbc.insets = new Insets(8,0,2,0); clockCtrlPanel.add(scLabel,cgbc);
        cgbc.insets = new Insets(3,0,3,0);

        shotClockLabel = new JLabel("24", SwingConstants.CENTER);
        shotClockLabel.setFont(FONT_DIGITAL_MEDIUM);
        shotClockLabel.setForeground(NEW_SHOT_CLOCK_FG);
        shotClockLabel.setOpaque(true);
        shotClockLabel.setBackground(NEW_DISPLAY_BOX_BG.darker());
        shotClockLabel.setBorder(new LineBorder(NEW_SHOT_CLOCK_FG.darker()));
        cgbc.gridy = 3; clockCtrlPanel.add(shotClockLabel, cgbc);

        JPanel shotClockBtns = new JPanel(new GridLayout(1,3,5,0));
        shotClockBtns.setOpaque(false);
        startShotClockButton = createStyledDarkButton("S", NEW_MAIN_TIMER_FG, NEW_BUTTON_BG, e -> startShotClock());
        shotClockBtns.add(startShotClockButton);
        shotClockBtns.add(createStyledDarkButton("B", NEW_SHOT_CLOCK_FG, NEW_BUTTON_BG, e -> playVisualBuzzer(shotClockLabel)));
        shotClockBtns.add(createStyledDarkButton("R", NEW_TEAM2_ACCENT, NEW_BUTTON_BG, e -> resetShotClock()));
        cgbc.gridy = 4; clockCtrlPanel.add(shotClockBtns, cgbc);

        gbc.gridx = 1; gbc.gridy = 0; gbc.weightx = 0.4;
        panel.add(clockCtrlPanel, gbc);

        JPanel team2CtrlPanel = createSingleTeamControls(2, "Team 2", NEW_TEAM2_ACCENT);
        gbc.gridx = 2; gbc.gridy = 0; gbc.weightx = 0.3;
        gbc.fill = GridBagConstraints.BOTH;
        panel.add(team2CtrlPanel, gbc);
        gbc.fill = GridBagConstraints.HORIZONTAL;

        JButton clearDataBtn = createStyledDarkButton("RESET ALL DATA", NEW_TEXT_COLOR_PRIMARY, NEW_TEAM2_ACCENT.darker(), e -> clearAllGameData());
        clearDataBtn.setFont(FONT_SANS_BUTTON.deriveFont(Font.BOLD, 16f));
        gbc.gridx = 1; gbc.gridy = 1; gbc.weightx = 0; gbc.anchor = GridBagConstraints.CENTER;
        gbc.fill = GridBagConstraints.HORIZONTAL;
        panel.add(clearDataBtn, gbc);

        return panel;
    }

    private JPanel createSingleTeamControls(int teamId, String teamLabel, Color accentColor) {
        JPanel panel = new JPanel(new GridBagLayout());
        panel.setOpaque(false);
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.insets = new Insets(3,3,3,3);
        gbc.weightx = 1.0;
        gbc.gridwidth = GridBagConstraints.REMAINDER;

        JLabel nameLabel = new JLabel(teamLabel.toUpperCase(), SwingConstants.CENTER);
        nameLabel.setFont(FONT_SANS_MEDIUM_LABEL.deriveFont(Font.BOLD));
        nameLabel.setForeground(accentColor);
        gbc.gridy = 0; panel.add(nameLabel, gbc);

        JPanel scoreBtns = new JPanel(new GridLayout(1,4,3,0));
        scoreBtns.setOpaque(false);
        scoreBtns.add(createStyledDarkButton("-1", accentColor.brighter(), NEW_BUTTON_BG, e -> updateScore(teamId, -1)));
        scoreBtns.add(createStyledDarkButton("+1", accentColor.brighter(), NEW_BUTTON_BG, e -> updateScore(teamId, 1)));
        scoreBtns.add(createStyledDarkButton("+2", accentColor.brighter(), NEW_BUTTON_BG, e -> updateScore(teamId, 2)));
        scoreBtns.add(createStyledDarkButton("+3", accentColor.brighter(), NEW_BUTTON_BG, e -> updateScore(teamId, 3)));
        gbc.gridy = 1; panel.add(scoreBtns, gbc);

        JPanel foulBtns = new JPanel(new GridLayout(1,2,3,0));
        foulBtns.setOpaque(false);
        foulBtns.add(createStyledDarkButton("Foul +", accentColor.brighter(), NEW_BUTTON_BG, e -> updateFouls(teamId, 1)));
        foulBtns.add(createStyledDarkButton("Foul -", accentColor.brighter(), NEW_BUTTON_BG, e -> updateFouls(teamId, -1)));
        gbc.gridy = 2; panel.add(foulBtns, gbc);

        JPanel timeoutBtns = new JPanel(new GridLayout(1,2,3,0));
        timeoutBtns.setOpaque(false);
        timeoutBtns.add(createStyledDarkButton("Timeout -", accentColor.brighter(), NEW_BUTTON_BG, e -> updateTimeouts(teamId, -1)));
        timeoutBtns.add(createStyledDarkButton("Timeout +", accentColor.brighter(), NEW_BUTTON_BG, e -> updateTimeouts(teamId, 1)));
        gbc.gridy = 3; panel.add(timeoutBtns, gbc);

        gbc.gridy = 4; gbc.weighty = 1.0;
        panel.add(Box.createVerticalGlue(), gbc);

        return panel;
    }

    private JRadioButton createStyledDarkRadioButton(String text, ActionListener actionListener) {
        JRadioButton radioButton = new JRadioButton(text);
        radioButton.setFont(FONT_SANS_RADIO);
        radioButton.setOpaque(false);
        radioButton.setForeground(NEW_TEXT_COLOR_PRIMARY);
        radioButton.addActionListener(actionListener);
        radioButton.setFocusPainted(false);
        return radioButton;
    }

    private JButton createStyledDarkButton(String text, Color fgColor, Color bgColor, ActionListener actionListener) {
        JButton button = new JButton(text);
        button.setFont(FONT_SANS_BUTTON);
        button.setForeground(fgColor);
        button.setBackground(bgColor);
        button.setOpaque(true);
        button.setFocusPainted(false);
        button.setBorder(new CompoundBorder(
                new LineBorder(fgColor.darker(), 1),
                new EmptyBorder(8, 12, 8, 12)
        ));
        if (actionListener != null) {
            button.addActionListener(actionListener);
        }
        return button;
    }

    private void initializeUIData() {
        updateScoreLabels();
        updateFoulMarkerLabels();
        updateTimeoutsDisplayLabels();
        updateMainTimerLabel();
        updateShotClockLabel();
        if (team1NameField != null) team1NameField.setText("HOME");
        if (team2NameField != null) team2NameField.setText("GUEST");
    }

    private void updateScoreLabels() {
        if (score1Label != null) score1Label.setText(String.format("%02d", score1));
        if (score2Label != null) score2Label.setText(String.format("%02d", score2));
    }

    private void updateFoulMarkerLabels() {
        updateSingleFoulMarkerLabelSet(team1FoulMarkerLabels, fouls1, NEW_TEAM1_ACCENT);
        updateSingleFoulMarkerLabelSet(team2FoulMarkerLabels, fouls2, NEW_TEAM2_ACCENT);
    }

    private void updateSingleFoulMarkerLabelSet(JLabel[] markers, int foulCount, Color activeColor) {
        if (markers == null) return;
        for (int i = 0; i < MAX_FOULS_DISPLAY; i++) {
            if (markers[i] == null) continue;
            if (i < foulCount) {
                markers[i].setText(FOUL_ACTIVE_CHAR);
                markers[i].setForeground(activeColor);
            } else {
                markers[i].setText(FOUL_INACTIVE_CHAR);
                markers[i].setForeground(NEW_FOUL_MARKER_INACTIVE_FG);
            }
        }
    }

    private void updateTimeoutsDisplayLabels() {
        if(timeouts1DisplayLabel != null) timeouts1DisplayLabel.setText(String.valueOf(timeouts1));
        if(timeouts2DisplayLabel != null) timeouts2DisplayLabel.setText(String.valueOf(timeouts2));
    }

    private void updateMainTimerLabel() {
        if (mainTimerLabel == null) return;
        int currentSeconds = Math.max(0, mainTimerSeconds);
        int minutes = currentSeconds / 60;
        int seconds = currentSeconds % 60;
        mainTimerLabel.setText(String.format("%02d:%02d", minutes, seconds));
    }

    private void updateShotClockLabel() {
        if (shotClockLabel != null) shotClockLabel.setText(String.format("%02d", Math.max(0, shotClockSeconds)));
    }

    private void updateScore(int team, int points) {
        if (team == 1) {
            score1 = Math.max(0, score1 + points);
        } else {
            score2 = Math.max(0, score2 + points);
        }
        updateScoreLabels();
    }

    private void updateFouls(int team, int amount) {
        if (team == 1) {
            fouls1 += amount;
            if (fouls1 < 0) fouls1 = 0;
            if (fouls1 > MAX_FOULS_DISPLAY * 2) fouls1 = MAX_FOULS_DISPLAY * 2;
        } else {
            fouls2 += amount;
            if (fouls2 < 0) fouls2 = 0;
            if (fouls2 > MAX_FOULS_DISPLAY * 2) fouls2 = MAX_FOULS_DISPLAY * 2;
        }
        updateFoulMarkerLabels();
    }

    private void updateTimeouts(int team, int change) {
        if (team == 1) {
            timeouts1 = Math.max(0, Math.min(INITIAL_TIMEOUTS + 2, timeouts1 + change));
        } else {
            timeouts2 = Math.max(0, Math.min(INITIAL_TIMEOUTS + 2, timeouts2 + change));
        }
        updateTimeoutsDisplayLabels();
    }

    private void setMainTimerDurationAction(int totalSeconds) {
        pauseMainTimer();
        mainTimerSeconds = totalSeconds;
        mainTimerCountsUp = false;
        updateMainTimerLabel();
        if (startPauseMainTimerButton != null) startPauseMainTimerButton.setText("START");
    }

    private void toggleMainTimer() {
        if (mainTimerRunning) {
            pauseMainTimer();
        } else {
            if (mainTimerSeconds <= 0 && !mainTimerCountsUp && timerPresetGroup.getSelection() != null) {
                if (radio5min.isSelected()) setMainTimerDurationAction(5 * 60);
                else if (radio10min.isSelected()) setMainTimerDurationAction(10 * 60);
                else if (radio12min.isSelected()) setMainTimerDurationAction(12 * 60);
            }
            startMainTimer();
        }
    }

    private void startMainTimer() {
        if (mainTimerRunning && mainTimer != null && mainTimer.isRunning()) return;
        if (mainTimerSeconds <= 0 && !mainTimerCountsUp) return;

        mainTimerRunning = true;
        if (startPauseMainTimerButton != null) startPauseMainTimerButton.setText("PAUSE");
        if (mainTimer == null) {
            mainTimer = new Timer(1000, e -> {
                if (mainTimerCountsUp) {
                    mainTimerSeconds++;
                } else {
                    mainTimerSeconds--;
                    if (mainTimerSeconds < 0) {
                        mainTimerSeconds = 0;
                        pauseMainTimer();
                        if (mainTimerLabel != null) playVisualBuzzer(mainTimerLabel);
                        return;
                    }
                }
                updateMainTimerLabel();
            });
            mainTimer.setInitialDelay(0);
        }
        mainTimer.start();
    }

    private void pauseMainTimer() {
        if (mainTimer != null) {
            mainTimer.stop();
        }
        mainTimerRunning = false;
        if (startPauseMainTimerButton != null) startPauseMainTimerButton.setText("START");
    }

    private void resetMainTimer() {
        pauseMainTimer();
        boolean presetApplied = false;
        if (timerPresetGroup != null && timerPresetGroup.getSelection() != null) {
            if (radio5min.isSelected()) {
                setMainTimerDurationAction(5 * 60); presetApplied = true;
            } else if (radio10min.isSelected()) {
                setMainTimerDurationAction(10 * 60); presetApplied = true;
            } else if (radio12min.isSelected()) {
                setMainTimerDurationAction(12 * 60); presetApplied = true;
            }
        }

        if (!presetApplied) {
            mainTimerSeconds = 0;
            mainTimerCountsUp = true;
            updateMainTimerLabel();
        }

        if (startPauseMainTimerButton != null) {
            startPauseMainTimerButton.setText("START");
        }
    }

    private void startShotClock() {
        if (shotClockRunning || shotClockSeconds <= 0) return;
        shotClockRunning = true;
        if (startShotClockButton != null) startShotClockButton.setEnabled(false);
        if (shotClockTimer == null) {
            shotClockTimer = new Timer(1000, e -> {
                shotClockSeconds--;
                updateShotClockLabel();
                if (shotClockSeconds <= 0) {
                    stopShotClockTimer();
                    if (shotClockLabel != null) playVisualBuzzer(shotClockLabel);
                    shotClockSeconds = 0;
                    updateShotClockLabel();
                }
            });
            shotClockTimer.setInitialDelay(0);
        }
        shotClockTimer.start();
    }

    private void stopShotClockTimer() {
        if (shotClockTimer != null) {
            shotClockTimer.stop();
        }
        shotClockRunning = false;
        if (startShotClockButton != null) startShotClockButton.setEnabled(true);
    }

    private void resetShotClock() {
        stopShotClockTimer();
        shotClockSeconds = DEFAULT_SHOT_CLOCK_DURATION;
        updateShotClockLabel();
    }

    private void playVisualBuzzer(JLabel labelToFlash) {
        if (labelToFlash == null) return;

        if (hornSoundClip != null) {
            if (hornSoundClip.isRunning()) {
                hornSoundClip.stop();
            }
            hornSoundClip.setFramePosition(0);
            hornSoundClip.start();
        } else {
            Toolkit.getDefaultToolkit().beep();
        }

        Color originalColor = labelToFlash.getForeground();
        Color originalBg = labelToFlash.getBackground();
        final Color flashColor = (labelToFlash == mainTimerLabel) ? NEW_MAIN_TIMER_FG.darker() : NEW_SHOT_CLOCK_FG.darker();

        Timer flashTimer = new Timer(150, new ActionListener() {
            private int count = 0;
            private boolean isFlashed = false;
            @Override
            public void actionPerformed(ActionEvent e) {
                if (isFlashed) {
                    labelToFlash.setForeground(originalColor);
                    labelToFlash.setBackground(originalBg);
                } else {
                    labelToFlash.setForeground(Color.RED);
                    labelToFlash.setBackground(flashColor.brighter());
                }
                isFlashed = !isFlashed;
                count++;
                if (count >= 10) {
                    ((Timer) e.getSource()).stop();
                    labelToFlash.setForeground(originalColor);
                    labelToFlash.setBackground(originalBg);
                }
            }
        });
        flashTimer.start();
    }

    private void clearAllGameData() {
        pauseMainTimer();
        stopShotClockTimer();

        score1 = 0; score2 = 0; updateScoreLabels();
        fouls1 = 0; fouls2 = 0; updateFoulMarkerLabels();

        timeouts1 = INITIAL_TIMEOUTS; timeouts2 = INITIAL_TIMEOUTS;
        updateTimeoutsDisplayLabels();

        if (radio5min != null) radio5min.setSelected(true);
        setMainTimerDurationAction(5 * 60);

        resetShotClock();

        if (team1NameField != null) team1NameField.setText("HOME");
        if (team2NameField != null) team2NameField.setText("GUEST");

        JOptionPane.showMessageDialog(this, "All Game Data Reset!", "System Reset", JOptionPane.INFORMATION_MESSAGE);
    }
}
