package blibioteca;

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.Random;

public class operadoreslogicos extends JPanel implements KeyListener, MouseMotionListener, MouseListener {

    // Dimensões da tela
    private static final int LARGURA = 800;
    private static final int ALTURA = 600;

    // Estados do jogo
    private enum EstadoJogo {
        MENU_MODO, MENU_TIPO_JOGO, MENU_META_PONTOS, MENU_OPCOES,
        JOGANDO, PAUSADO, FIM_DE_JOGO
    }

    private EstadoJogo estadoAtual = EstadoJogo.MENU_MODO;
    private EstadoJogo estadoAnteriorOpcoes = EstadoJogo.MENU_MODO;

    // Configurações
    private boolean modoBot = false;
    private boolean modoTreino = false;
    private boolean temPartidaSalva = false;

    private enum TipoJogo { PONTOS_DEFINIDOS, INFINITO }
    private TipoJogo tipoJogoAtual = TipoJogo.PONTOS_DEFINIDOS;

    private enum Dificuldade { FACIL, MEDIO, DIFICIL, FRENESI }
    private Dificuldade dificuldadeAtual = Dificuldade.MEDIO;
    private final String[] NOMES_DIFICULDADE = {"Fácil", "Médio", "Difícil", "Frenesi"};

    private final int[] OPCOES_PONTOS = {3, 5, 10, 15, 20};
    private int indiceOpcaoPontos = 1;
    private int pontosParaVencer = 5;

    // Seleções de menus
    private int opcaoMenuModo = 0;
    private int opcaoMenuTipoJogo = 0;
    private int opcaoMenuOpcoes = 0;
    private int opcaoMenuPausa = 0;

    // Mapeamento de teclas
    private int teclaJ1Cima = KeyEvent.VK_W;
    private int teclaJ1Baixo = KeyEvent.VK_S;
    private int teclaJ2Cima = KeyEvent.VK_UP;
    private int teclaJ2Baixo = KeyEvent.VK_DOWN;
    private boolean aguardandoReinstalaTecla = false;
    private String acaoMapeando = "";

    // Mouse
    private enum ModoMouse { DESATIVADO, PLAYER_1, PLAYER_2 }
    private ModoMouse modoMouseAtual = ModoMouse.DESATIVADO;
    private final String[] NOMES_MODO_MOUSE = {"Nenhum (Desativado)", "Jogador 1", "Jogador 2"};

    // Cores
    private final Color[] CORES_DISPONIVEIS = {
        Color.WHITE, Color.RED, Color.GREEN, Color.BLUE,
        Color.YELLOW, Color.CYAN, Color.MAGENTA, Color.ORANGE
    };

    private final String[] NOMES_CORES = {
        "Branco", "Vermelho", "Verde", "Azul",
        "Amarelo", "Ciano", "Magenta", "Laranja"
    };

    private int idxCorJ1 = 0;
    private int idxCorJ2 = 0;
    private int idxCorBola = 0;

    private Color corJ1 = Color.WHITE;
    private Color corJ2 = Color.WHITE;
    private Color corBola = Color.WHITE;

    // Dimensões customizáveis
    private final int[] OPCOES_TAM_RAQUETE = {60, 100, 140};
    private final String[] NOMES_TAM_RAQUETE = {
        "Pequena (60px)", "Média (100px)", "Grande (140px)"
    };
    private int idxTamRaquete = 1;
    private int alturaRaquete = 100;

    private final int[] OPCOES_TAM_BOLA = {12, 20, 30};
    private final String[] NOMES_TAM_BOLA = {
        "Pequena (12px)", "Média (20px)", "Grande (30px)"
    };
    private int idxTamBola = 1;
    private int tamanhoBola = 20;

    private final int[] OPCOES_VEL_RAQUETE = {4, 6, 9};
    private final String[] NOMES_VEL_RAQUETE = {
        "Lenta (4px)", "Média (6px)", "Rápida (9px)"
    };
    private int idxVelRaquete = 1;
    private int velocidadeRaquete = 6;

    private static final int LARGURA_RAQUETE = 15;

    // Velocidades e física
    private double velocidadeBolaAtual = 4.0;
    private double velocidadeBolaBase = 4.0;
    private int velocidadeBot = 4;
    private boolean bolaEsperandoInicio = false;

    private double j1Y = 250, j2Y = 250;
    private double bolaX = 400, bolaY = 300;
    private double bolaXDir = 4, bolaYDir = 4;

    private int pontosJ1 = 0, pontosJ2 = 0;
    private int rebatesTreinoAtual = 0;
    private int recordeTreino = 0;

    private boolean j1Cima, j1Baixo, j2Cima, j2Baixo;

    private final Random random = new Random();
    private final Timer gameTimer;

    public operadoreslogicos() {
        setPreferredSize(new Dimension(LARGURA, ALTURA));
        setBackground(Color.BLACK);
        setFocusable(true);
        setDoubleBuffered(true);

        addKeyListener(this);
        addMouseMotionListener(this);
        addMouseListener(this);

        gameTimer = new Timer(10, e -> {
            atualizar();
            repaint();
        });
        gameTimer.start();

        requestFocusInWindow();
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);

        switch (estadoAtual) {
            case MENU_MODO -> desenharMenuModo(g);
            case MENU_TIPO_JOGO -> desenharMenuTipoJogo(g);
            case MENU_META_PONTOS -> desenharMenuMetaPontos(g);
            case MENU_OPCOES -> desenharMenuOpcoes(g);
            case JOGANDO -> desenharJogo(g);
            case PAUSADO -> {
                desenharJogo(g);
                desenharMenuPausa(g);
            }
            case FIM_DE_JOGO -> desenharFimDeJogo(g);
        }
    }

    private void desenharMenuModo(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 48));
        g.drawString("PONG GAME", LARGURA / 2 - 150, 110);

        Font font = new Font("Arial", Font.PLAIN, 22);
        String[] ops = {
            temPartidaSalva ? "Continuar Partida Anterior" : "[Sem Partida Salva]",
            "1 Jogador (vs Bot)",
            "2 Jogadores",
            "Modo Treino (Solo)",
            "Opções do Jogo"
        };

        for (int i = 0; i < ops.length; i++) {
            if (i == 0 && !temPartidaSalva) {
                g.setColor(Color.GRAY);
            } else {
                g.setColor(opcaoMenuModo == i ? Color.YELLOW : Color.WHITE);
            }

            String prefixo = opcaoMenuModo == i ? "> " : "   ";
            g.setFont(font);
            g.drawString(prefixo + ops[i], LARGURA / 2 - 180, 180 + (i * 45));
        }

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 14));
        g.drawString(
            "Use W/S, SETAS ou MOUSE para navegar | ESPAÇO ou CLIQUE para confirmar",
            LARGURA / 2 - 240, 480
        );
    }

    private void desenharMenuOpcoes(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 36));
        g.drawString("OPÇÕES DO JOGO", LARGURA / 2 - 180, 50);

        Font font = new Font("Arial", Font.PLAIN, 14);
        String[] ops = {
            "Dificuldade: < " + NOMES_DIFICULDADE[dificuldadeAtual.ordinal()] + " >",
            "Cor Raquete J1: < " + NOMES_CORES[idxCorJ1] + " >",
            "Cor Raquete J2 / Parede: < " + NOMES_CORES[idxCorJ2] + " >",
            "Cor da Bolinha: < " + NOMES_CORES[idxCorBola] + " >",
            "Tamanho Raquete: < " + NOMES_TAM_RAQUETE[idxTamRaquete] + " >",
            "Tamanho Bola: < " + NOMES_TAM_BOLA[idxTamBola] + " >",
            "Velocidade Raquete: < " + NOMES_VEL_RAQUETE[idxVelRaquete] + " >",
            "Controle por Mouse: < " + NOMES_MODO_MOUSE[modoMouseAtual.ordinal()] + " >",
            "Mapear J1 Subir: [ " + teclaParaTexto(teclaJ1Cima) + " ]",
            "Mapear J1 Descer: [ " + teclaParaTexto(teclaJ1Baixo) + " ]",
            "Mapear J2 Subir: [ " + teclaParaTexto(teclaJ2Cima) + " ]",
            "Mapear J2 Descer: [ " + teclaParaTexto(teclaJ2Baixo) + " ]",
            "Voltar"
        };

        for (int i = 0; i < ops.length; i++) {
            g.setColor(opcaoMenuOpcoes == i ? Color.YELLOW : Color.WHITE);
            String prefixo = opcaoMenuOpcoes == i ? "> " : "   ";
            g.setFont(font);
            g.drawString(prefixo + ops[i], 80, 85 + (i * 30));
        }

        g.setColor(corJ1);
        g.fillRect(560, 102, 18, 18);

        g.setColor(corJ2);
        g.fillRect(560, 132, 18, 18);

        g.setColor(corBola);
        g.fillOval(560, 162, 18, 18);

        if (aguardandoReinstalaTecla) {
            g.setColor(new Color(0, 0, 0, 200));
            g.fillRect(150, 200, 500, 150);

            g.setColor(Color.YELLOW);
            g.drawRect(150, 200, 500, 150);
            g.setFont(new Font("Arial", Font.BOLD, 16));
            g.drawString(
                "Pressione a nova tecla para: " + acaoMapeando,
                170, 250
            );
        }

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 12));
        g.drawString(
            "Navegar: CIMA/BAIXO ou MOUSE | Alterar: ESQUERDA/DIREITA ou CLIQUE",
            150, 510
        );
        g.drawString(
            "Pressione ESPAÇO/ENTER ou ESC para confirmar/voltar",
            200, 535
        );
    }

    private void desenharMenuTipoJogo(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("Tipo de Partida", LARGURA / 2 - 140, 170);

        Font font = new Font("Arial", Font.PLAIN, 22);
        String[] ops = {"Partida por Pontos", "Modo Infinito"};

        for (int i = 0; i < ops.length; i++) {
            g.setColor(opcaoMenuTipoJogo == i ? Color.YELLOW : Color.WHITE);
            String prefixo = opcaoMenuTipoJogo == i ? "> " : "   ";
            g.setFont(font);
            g.drawString(prefixo + ops[i], LARGURA / 2 - 120, 260 + (i * 60));
        }
    }

    private void desenharMenuMetaPontos(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("Meta de Pontos", LARGURA / 2 - 150, 130);

        Font font = new Font("Arial", Font.PLAIN, 20);

        for (int i = 0; i < OPCOES_PONTOS.length; i++) {
            g.setColor(indiceOpcaoPontos == i ? Color.YELLOW : Color.WHITE);
            String prefixo = indiceOpcaoPontos == i ? "> " : "   ";
            g.setFont(font);
            g.drawString(
                prefixo + OPCOES_PONTOS[i] + " Pontos",
                LARGURA / 2 - 80,
                200 + (i * 45)
            );
        }
    }

    private void desenharJogo(Graphics g) {
        if (!modoTreino) {
            g.setColor(Color.WHITE);
            for (int i = 0; i < ALTURA; i += 30) {
                g.fillRect(LARGURA / 2 - 2, i, 4, 15);
            }

            g.setColor(corJ2);
            g.fillRect(
                LARGURA - 30 - LARGURA_RAQUETE,
                (int) j2Y,
                LARGURA_RAQUETE,
                alturaRaquete
            );
        } else {
            g.setColor(corJ2);
            g.fillRect(LARGURA - 10, 0, 10, ALTURA);
        }

        g.setColor(corJ1);
        g.fillRect(30, (int) j1Y, LARGURA_RAQUETE, alturaRaquete);

        g.setColor(corBola);
        g.fillOval((int) bolaX, (int) bolaY, tamanhoBola, tamanhoBola);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 20));

        if (modoTreino) {
            g.drawString("Rebates: " + rebatesTreinoAtual, 180, 60);
            g.drawString("Recorde: " + recordeTreino, 500, 60);
        } else {
            g.drawString("Jogador 1: " + pontosJ1, 160, 60);
            String nomeJ2 = modoBot ? "Bot: " : "Jogador 2: ";
            g.drawString(nomeJ2 + pontosJ2, 500, 60);
        }

        if (bolaEsperandoInicio) {
            g.setColor(Color.YELLOW);
            g.setFont(new Font("Arial", Font.BOLD, 16));
            g.drawString(
                "Pressione qualquer tecla ou clique com o mouse para iniciar!",
                LARGURA / 2 - 240, 120
            );
        }
    }

    private void desenharMenuPausa(Graphics g) {
        g.setColor(new Color(0, 0, 0, 180));
        g.fillRect(0, 0, LARGURA, ALTURA);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("PAUSA", LARGURA / 2 - 70, 180);

        Font font = new Font("Arial", Font.PLAIN, 22);
        String[] ops = {
            "Continuar Jogo",
            "Opções do Jogo",
            "Resetar Pontos",
            "Voltar ao Menu Principal"
        };

        for (int i = 0; i < ops.length; i++) {
            g.setColor(opcaoMenuPausa == i ? Color.YELLOW : Color.WHITE);
            String prefixo = opcaoMenuPausa == i ? "> " : "   ";
            g.setFont(font);
            g.drawString(prefixo + ops[i], LARGURA / 2 - 140, 250 + (i * 50));
        }
    }

    private void desenharFimDeJogo(Graphics g) {
        g.setColor(Color.RED);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("FIM DE JOGO", LARGURA / 2 - 130, 220);

        String vencedor;
        if (pontosJ1 >= pontosParaVencer) {
            vencedor = "Jogador 1 Venceu!";
        } else {
            vencedor = modoBot ? "O Bot Venceu!" : "Jogador 2 Venceu!";
        }

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 26));
        g.drawString(vencedor, LARGURA / 2 - 120, 310);

        g.setFont(new Font("Arial", Font.ITALIC, 16));
        g.drawString(
            "Pressione ESPAÇO, ESC ou CLIQUE para voltar ao Menu",
            LARGURA / 2 - 200, 410
        );
    }

    private void aplicarDificuldade() {
        switch (dificuldadeAtual) {
            case FACIL -> {
                velocidadeBolaBase = 3.0;
                velocidadeBot = 2;
            }
            case MEDIO -> {
                velocidadeBolaBase = 5.0;
                velocidadeBot = 4;
            }
            case DIFICIL -> {
                velocidadeBolaBase = 7.0;
                velocidadeBot = 6;
            }
            case FRENESI -> {
                velocidadeBolaBase = 3.0;
                velocidadeBot = 6;
            }
        }

        double velocidadeAntiga = velocidadeBolaAtual;
        velocidadeBolaAtual = velocidadeBolaBase;

        if (velocidadeAntiga != 0) {
            double fatorProporcao = velocidadeBolaAtual / velocidadeAntiga;
            bolaXDir *= fatorProporcao;
            bolaYDir *= fatorProporcao;
        }
    }

    private void aumentarVelocidadeFrenesi() {
        if (dificuldadeAtual == Dificuldade.FRENESI && velocidadeBolaAtual < 14.0) {
            velocidadeBolaAtual += 0.5;

            double sinalX = bolaXDir > 0 ? 1 : -1;
            double sinalY = bolaYDir > 0 ? 1 : -1;

            bolaXDir = sinalX * velocidadeBolaAtual;
            bolaYDir = sinalY * velocidadeBolaAtual;
        }
    }

    private void atualizar() {
        if (estadoAtual != EstadoJogo.JOGANDO) return;

        Point mouse = getMousePosition();

        // Movimento do Jogador 1
        if (modoMouseAtual == ModoMouse.PLAYER_1 && mouse != null) {
            j1Y = mouse.y - (alturaRaquete / 2.0);
        } else {
            if (j1Cima) j1Y -= velocidadeRaquete;
            if (j1Baixo) j1Y += velocidadeRaquete;
        }

        // Limites J1
        if (j1Y < 0) j1Y = 0;
        if (j1Y > ALTURA - alturaRaquete) {
            j1Y = ALTURA - alturaRaquete;
        }

        // Movimento Jogador 2 / Bot
        if (!modoTreino) {
            if (modoBot) {
                if (bolaXDir > 0) {
                    double centroRaquete = j2Y + alturaRaquete / 2.0;
                    double centroBola = bolaY + tamanhoBola / 2.0;

                    if (centroRaquete < centroBola && j2Y < ALTURA - alturaRaquete) {
                        j2Y += velocidadeBot;
                    } else if (centroRaquete > centroBola && j2Y > 0) {
                        j2Y -= velocidadeBot;
                    }
                }
            } else {
                if (modoMouseAtual == ModoMouse.PLAYER_2 && mouse != null) {
                    j2Y = mouse.y - (alturaRaquete / 2.0);
                } else {
                    if (j2Cima) j2Y -= velocidadeRaquete;
                    if (j2Baixo) j2Y += velocidadeRaquete;
                }
            }

            if (j2Y < 0) j2Y = 0;
            if (j2Y > ALTURA - alturaRaquete) {
                j2Y = ALTURA - alturaRaquete;
            }
        }

        // Movimento da bola
        if (!bolaEsperandoInicio) {
            bolaX += bolaXDir;
            bolaY += bolaYDir;

            // Colisão topo/base
            if (bolaY <= 0 || bolaY >= ALTURA - tamanhoBola) {
                bolaYDir = -bolaYDir;
            }

            // Colisão raquete J1
            if (bolaX <= 30 + LARGURA_RAQUETE && bolaX >= 30) {
                if (bolaY + tamanhoBola >= j1Y && bolaY <= j1Y + alturaRaquete) {
                    bolaXDir = Math.abs(bolaXDir);
                    bolaX = 30 + LARGURA_RAQUETE + 1;
                    aumentarVelocidadeFrenesi();

                    if (modoTreino) {
                        rebatesTreinoAtual++;
                        if (rebatesTreinoAtual > recordeTreino) {
                            recordeTreino = rebatesTreinoAtual;
                        }
                    }
                }
            }

            // Colisão lado direito
            if (modoTreino) {
                if (bolaX + tamanhoBola >= LARGURA - 10) {
                    bolaXDir = -Math.abs(bolaXDir);
                    bolaX = LARGURA - 10 - tamanhoBola - 1;
                    aumentarVelocidadeFrenesi();
                }
            } else {
                if (bolaX + tamanhoBola >= LARGURA - 30 - LARGURA_RAQUETE
                    && bolaX + tamanhoBola <= LARGURA - 30) {

                    if (bolaY + tamanhoBola >= j2Y && bolaY <= j2Y + alturaRaquete) {
                        bolaXDir = -Math.abs(bolaXDir);
                        bolaX = LARGURA - 30 - LARGURA_RAQUETE - tamanhoBola - 1;
                        aumentarVelocidadeFrenesi();
                    }
                }
            }

            // Pontuação
            if (bolaX < 0) {
                if (modoTreino) {
                    rebatesTreinoAtual = 0;
                } else {
                    pontosJ2++;
                    verificarVitoria();
                }
                reiniciarBola();
            } else if (bolaX > LARGURA && !modoTreino) {
                pontosJ1++;
                verificarVitoria();
                reiniciarBola();
            }
        }
    }

    private void verificarVitoria() {
        if (tipoJogoAtual == TipoJogo.PONTOS_DEFINIDOS) {
            if (pontosJ1 >= pontosParaVencer || pontosJ2 >= pontosParaVencer) {
                estadoAtual = EstadoJogo.FIM_DE_JOGO;
                temPartidaSalva = false;
            }
        }
    }

    private void reiniciarBola() {
        bolaX = LARGURA / 2.0 - tamanhoBola / 2.0;
        bolaY = ALTURA / 2.0 - tamanhoBola / 2.0;

        velocidadeBolaAtual = velocidadeBolaBase;

        double angulo;

        if (modoTreino) {
            angulo = (random.nextDouble() * Math.PI / 2.0) - (Math.PI / 4.0);
        } else {
            boolean paraDireita = random.nextInt(2) == 0;
            double variacao = (random.nextDouble() * Math.PI / 2.0) - (Math.PI / 4.0);
            angulo = paraDireita ? variacao : Math.PI + variacao;
        }

        bolaXDir = velocidadeBolaAtual * Math.cos(angulo);
        bolaYDir = velocidadeBolaAtual * Math.sin(angulo);

        if (!modoBot) {
            bolaEsperandoInicio = true;
        }
    }

    private void resetarPartida(boolean zerarPontos) {
        if (zerarPontos) {
            pontosJ1 = 0;
            pontosJ2 = 0;
            rebatesTreinoAtual = 0;
        }

        j1Y = (ALTURA - alturaRaquete) / 2.0;
        j2Y = (ALTURA - alturaRaquete) / 2.0;

        j1Cima = false;
        j1Baixo = false;
        j2Cima = false;
        j2Baixo = false;

        aplicarDificuldade();
        reiniciarBola();

        if (modoBot) {
            bolaEsperandoInicio = false;
        }

        temPartidaSalva = true;
    }

    private void alterarOpcaoOpcoes(int direcao) {
        switch (opcaoMenuOpcoes) {
            case 0 -> {
                int totalDif = Dificuldade.values().length;
                dificuldadeAtual = Dificuldade.values()[
                    (dificuldadeAtual.ordinal() + direcao + totalDif) % totalDif
                ];
                aplicarDificuldade();
            }

            case 1 -> {
                idxCorJ1 = (idxCorJ1 + direcao + CORES_DISPONIVEIS.length)
                    % CORES_DISPONIVEIS.length;
                corJ1 = CORES_DISPONIVEIS[idxCorJ1];
            }

            case 2 -> {
                idxCorJ2 = (idxCorJ2 + direcao + CORES_DISPONIVEIS.length)
                    % CORES_DISPONIVEIS.length;
                corJ2 = CORES_DISPONIVEIS[idxCorJ2];
            }

            case 3 -> {
                idxCorBola = (idxCorBola + direcao + CORES_DISPONIVEIS.length)
                    % CORES_DISPONIVEIS.length;
                corBola = CORES_DISPONIVEIS[idxCorBola];
            }

            case 4 -> {
                idxTamRaquete = (idxTamRaquete + direcao + OPCOES_TAM_RAQUETE.length)
                    % OPCOES_TAM_RAQUETE.length;
                alturaRaquete = OPCOES_TAM_RAQUETE[idxTamRaquete];
            }

            case 5 -> {
                idxTamBola = (idxTamBola + direcao + OPCOES_TAM_BOLA.length)
                    % OPCOES_TAM_BOLA.length;
                tamanhoBola = OPCOES_TAM_BOLA[idxTamBola];
            }

            case 6 -> {
                idxVelRaquete = (idxVelRaquete + direcao + OPCOES_VEL_RAQUETE.length)
                    % OPCOES_VEL_RAQUETE.length;
                velocidadeRaquete = OPCOES_VEL_RAQUETE[idxVelRaquete];
            }

            case 7 -> {
                int totalModos = ModoMouse.values().length;
                modoMouseAtual = ModoMouse.values()[
                    (modoMouseAtual.ordinal() + direcao + totalModos) % totalModos
                ];
            }

            case 8 -> {
                aguardandoReinstalaTecla = true;
                acaoMapeando = "J1 Subir";
            }

            case 9 -> {
                aguardandoReinstalaTecla = true;
                acaoMapeando = "J1 Descer";
            }

            case 10 -> {
                aguardandoReinstalaTecla = true;
                acaoMapeando = "J2 Subir";
            }

            case 11 -> {
                aguardandoReinstalaTecla = true;
                acaoMapeando = "J2 Descer";
            }

            case 12 -> estadoAtual = estadoAnteriorOpcoes;
        }
    }

    private String teclaParaTexto(int keyCode) {
        return KeyEvent.getKeyText(keyCode);
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int key = e.getKeyCode();

        if (aguardandoReinstalaTecla) {
            if (acaoMapeando.equals("J1 Subir")) teclaJ1Cima = key;
            else if (acaoMapeando.equals("J1 Descer")) teclaJ1Baixo = key;
            else if (acaoMapeando.equals("J2 Subir")) teclaJ2Cima = key;
            else if (acaoMapeando.equals("J2 Descer")) teclaJ2Baixo = key;

            aguardandoReinstalaTecla = false;
            repaint();
            return;
        }

        if (estadoAtual == EstadoJogo.MENU_MODO) {
            if (key == KeyEvent.VK_W || key == KeyEvent.VK_UP) {
                opcaoMenuModo = (opcaoMenuModo - 1 + 5) % 5;
            } else if (key == KeyEvent.VK_S || key == KeyEvent.VK_DOWN) {
                opcaoMenuModo = (opcaoMenuModo + 1) % 5;
            } else if (key == KeyEvent.VK_SPACE || key == KeyEvent.VK_ENTER) {
                confirmarMenuModo();
            }
        }

        else if (estadoAtual == EstadoJogo.MENU_TIPO_JOGO) {
            if (key == KeyEvent.VK_W || key == KeyEvent.VK_UP) {
                opcaoMenuTipoJogo = (opcaoMenuTipoJogo - 1 + 2) % 2;
            } else if (key == KeyEvent.VK_S || key == KeyEvent.VK_DOWN) {
                opcaoMenuTipoJogo = (opcaoMenuTipoJogo + 1) % 2;
            } else if (key == KeyEvent.VK_SPACE || key == KeyEvent.VK_ENTER) {
                confirmarMenuTipoJogo();
            } else if (key == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_MODO;
            }
        }

        else if (estadoAtual == EstadoJogo.MENU_META_PONTOS) {
            if (key == KeyEvent.VK_W || key == KeyEvent.VK_UP) {
                indiceOpcaoPontos =
                    (indiceOpcaoPontos - 1 + OPCOES_PONTOS.length) % OPCOES_PONTOS.length;
            } else if (key == KeyEvent.VK_S || key == KeyEvent.VK_DOWN) {
                indiceOpcaoPontos =
                    (indiceOpcaoPontos + 1) % OPCOES_PONTOS.length;
            } else if (key == KeyEvent.VK_SPACE || key == KeyEvent.VK_ENTER) {
                confirmarMenuMetaPontos();
            } else if (key == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_TIPO_JOGO;
            }
        }

        else if (estadoAtual == EstadoJogo.MENU_OPCOES) {
            if (key == KeyEvent.VK_W || key == KeyEvent.VK_UP) {
                opcaoMenuOpcoes =
                    (opcaoMenuOpcoes - 1 + 13) % 13;
            } else if (key == KeyEvent.VK_S || key == KeyEvent.VK_DOWN) {
                opcaoMenuOpcoes =
                    (opcaoMenuOpcoes + 1) % 13;
            } else if (key == KeyEvent.VK_D || key == KeyEvent.VK_RIGHT) {
                alterarOpcaoOpcoes(1);
            } else if (key == KeyEvent.VK_A || key == KeyEvent.VK_LEFT) {
                alterarOpcaoOpcoes(-1);
            } else if (key == KeyEvent.VK_ESCAPE) {
                estadoAtual = estadoAnteriorOpcoes;
            } else if (key == KeyEvent.VK_SPACE || key == KeyEvent.VK_ENTER) {
                alterarOpcaoOpcoes(1);
            }
        }

        else if (estadoAtual == EstadoJogo.JOGANDO) {
            if (key == KeyEvent.VK_ESCAPE) {
                opcaoMenuPausa = 0;
                estadoAtual = EstadoJogo.PAUSADO;
                return;
            }

            if (bolaEsperandoInicio) {
                bolaEsperandoInicio = false;
            }

            if (key == teclaJ1Cima) j1Cima = true;
            if (key == teclaJ1Baixo) j1Baixo = true;

            if (!modoBot && !modoTreino) {
                if (key == teclaJ2Cima) j2Cima = true;
                if (key == teclaJ2Baixo) j2Baixo = true;
            }
        }

        else if (estadoAtual == EstadoJogo.PAUSADO) {
            if (key == KeyEvent.VK_W || key == KeyEvent.VK_UP) {
                opcaoMenuPausa = (opcaoMenuPausa - 1 + 4) % 4;
            } else if (key == KeyEvent.VK_S || key == KeyEvent.VK_DOWN) {
                opcaoMenuPausa = (opcaoMenuPausa + 1) % 4;
            } else if (key == KeyEvent.VK_SPACE || key == KeyEvent.VK_ENTER) {
                confirmarMenuPausa();
            } else if (key == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.JOGANDO;
            }
        }

        else if (estadoAtual == EstadoJogo.FIM_DE_JOGO) {
            if (key == KeyEvent.VK_SPACE ||
                key == KeyEvent.VK_ENTER ||
                key == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_MODO;
            }
        }

        repaint();
    }

    @Override
    public void keyReleased(KeyEvent e) {
        int key = e.getKeyCode();

        if (key == teclaJ1Cima) j1Cima = false;
        if (key == teclaJ1Baixo) j1Baixo = false;
        if (key == teclaJ2Cima) j2Cima = false;
        if (key == teclaJ2Baixo) j2Baixo = false;
    }

    @Override
    public void keyTyped(KeyEvent e) {}

    // Métodos auxiliares de confirmação para reutilização com teclas e mouse
    private void confirmarMenuModo() {
        if (opcaoMenuModo == 0 && temPartidaSalva) {
            estadoAtual = EstadoJogo.JOGANDO;
        } else if (opcaoMenuModo == 4) {
            estadoAnteriorOpcoes = EstadoJogo.MENU_MODO;
            estadoAtual = EstadoJogo.MENU_OPCOES;
        } else if (opcaoMenuModo > 0) {
            modoBot = opcaoMenuModo == 1;
            modoTreino = opcaoMenuModo == 3;

            if (modoTreino) {
                resetarPartida(true);
                estadoAtual = EstadoJogo.JOGANDO;
            } else {
                estadoAtual = EstadoJogo.MENU_TIPO_JOGO;
            }
        }
    }

    private void confirmarMenuTipoJogo() {
        tipoJogoAtual = opcaoMenuTipoJogo == 0
            ? TipoJogo.PONTOS_DEFINIDOS
            : TipoJogo.INFINITO;

        if (tipoJogoAtual == TipoJogo.PONTOS_DEFINIDOS) {
            estadoAtual = EstadoJogo.MENU_META_PONTOS;
        } else {
            resetarPartida(true);
            estadoAtual = EstadoJogo.JOGANDO;
        }
    }

    private void confirmarMenuMetaPontos() {
        pontosParaVencer = OPCOES_PONTOS[indiceOpcaoPontos];
        resetarPartida(true);
        estadoAtual = EstadoJogo.JOGANDO;
    }

    private void confirmarMenuPausa() {
        if (opcaoMenuPausa == 0) {
            estadoAtual = EstadoJogo.JOGANDO;
        } else if (opcaoMenuPausa == 1) {
            estadoAnteriorOpcoes = EstadoJogo.PAUSADO;
            estadoAtual = EstadoJogo.MENU_OPCOES;
        } else if (opcaoMenuPausa == 2) {
            resetarPartida(true);
            estadoAtual = EstadoJogo.JOGANDO;
        } else if (opcaoMenuPausa == 3) {
            estadoAtual = EstadoJogo.MENU_MODO;
        }
    }

    @Override
    public void mouseMoved(MouseEvent e) {
        int mouseY = e.getY();

        if (estadoAtual == EstadoJogo.MENU_MODO) {
            for (int i = 0; i < 5; i++) {
                int topY = 155 + (i * 45);
                if (mouseY >= topY && mouseY <= topY + 35) {
                    opcaoMenuModo = i;
                    repaint();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_TIPO_JOGO) {
            for (int i = 0; i < 2; i++) {
                int topY = 235 + (i * 60);
                if (mouseY >= topY && mouseY <= topY + 45) {
                    opcaoMenuTipoJogo = i;
                    repaint();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_META_PONTOS) {
            for (int i = 0; i < OPCOES_PONTOS.length; i++) {
                int topY = 175 + (i * 45);
                if (mouseY >= topY && mouseY <= topY + 35) {
                    indiceOpcaoPontos = i;
                    repaint();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_OPCOES) {
            for (int i = 0; i < 13; i++) {
                int topY = 65 + (i * 30);
                if (mouseY >= topY && mouseY <= topY + 25) {
                    opcaoMenuOpcoes = i;
                    repaint();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.PAUSADO) {
            for (int i = 0; i < 4; i++) {
                int topY = 225 + (i * 50);
                if (mouseY >= topY && mouseY <= topY + 40) {
                    opcaoMenuPausa = i;
                    repaint();
                    break;
                }
            }
        }
    }

    @Override
    public void mouseDragged(MouseEvent e) {
        mouseMoved(e);
    }

    @Override
    public void mousePressed(MouseEvent e) {
        requestFocusInWindow();
        int mouseY = e.getY();

        if (estadoAtual == EstadoJogo.JOGANDO) {
            if (bolaEsperandoInicio) {
                bolaEsperandoInicio = false;
            }
        } else if (estadoAtual == EstadoJogo.MENU_MODO) {
            for (int i = 0; i < 5; i++) {
                int topY = 155 + (i * 45);
                if (mouseY >= topY && mouseY <= topY + 35) {
                    opcaoMenuModo = i;
                    confirmarMenuModo();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_TIPO_JOGO) {
            for (int i = 0; i < 2; i++) {
                int topY = 235 + (i * 60);
                if (mouseY >= topY && mouseY <= topY + 45) {
                    opcaoMenuTipoJogo = i;
                    confirmarMenuTipoJogo();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_META_PONTOS) {
            for (int i = 0; i < OPCOES_PONTOS.length; i++) {
                int topY = 175 + (i * 45);
                if (mouseY >= topY && mouseY <= topY + 35) {
                    indiceOpcaoPontos = i;
                    confirmarMenuMetaPontos();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_OPCOES) {
            alterarOpcaoOpcoes(
                e.getButton() == MouseEvent.BUTTON3 ? -1 : 1
            );
        } else if (estadoAtual == EstadoJogo.PAUSADO) {
            for (int i = 0; i < 4; i++) {
                int topY = 225 + (i * 50);
                if (mouseY >= topY && mouseY <= topY + 40) {
                    opcaoMenuPausa = i;
                    confirmarMenuPausa();
                    break;
                }
            }
        } else if (estadoAtual == EstadoJogo.FIM_DE_JOGO) {
            estadoAtual = EstadoJogo.MENU_MODO;
        }

        repaint();
    }

    @Override
    public void mouseClicked(MouseEvent e) {}

    @Override
    public void mouseReleased(MouseEvent e) {}

    @Override
    public void mouseEntered(MouseEvent e) {}

    @Override
    public void mouseExited(MouseEvent e) {}

    // Executável do jogo
    public static void main(String[] args) {
        JFrame window = new JFrame("Pong Game");
        operadoreslogicos gamePanel = new operadoreslogicos();
        
        window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        window.setResizable(false);
        window.add(gamePanel);
        window.pack();
        window.setLocationRelativeTo(null);
        window.setVisible(true);
    }
}
