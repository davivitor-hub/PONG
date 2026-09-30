Modifique aonde está escrito operadoreslogicos coloque o nome da sua class para funcionar corretamente esse jogo é facilmente atualizado com IA

package blibioteca;

import javax.swing.JFrame;
import javax.swing.JPanel;
import java.awt.Color;
import java.awt.Graphics;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.event.KeyListener;
import java.awt.event.KeyEvent;
import java.util.Random;

public class operadoreslogicos extends JPanel implements Runnable, KeyListener {
    // Dimensões da tela
    private static final int LARGURA = 800;
    private static final int ALTURA = 600;

    // Estados do Jogo
    private enum EstadoJogo { 
        MENU_MODO, MENU_TIPO_JOGO, MENU_META_PONTOS, MENU_OPCOES, 
        JOGANDO, PAUSADO, FIM_DE_JOGO 
    }
    private EstadoJogo estadoAtual = EstadoJogo.MENU_MODO;
    
    // Controle do estado de origem das Opções para retorno correto
    private EstadoJogo estadoAnteriorOpcoes = EstadoJogo.MENU_MODO;

    // Modos, Tipos de Jogo e Dificuldades
    private boolean modoBot = false;
    private boolean modoTreino = false;
    private boolean temPartidaSalva = false;

    private enum TipoJogo { PONTOS_DEFINIDOS, INFINITO }
    private TipoJogo tipoJogoAtual = TipoJogo.PONTOS_DEFINIDOS;

    private enum Dificuldade { FACIL, MEDIO, DIFICIL, FRENESI }
    private Dificuldade dificuldadeAtual = Dificuldade.MEDIO;
    private final String[] NOMES_DIFICULDADE = {"Fácil", "Médio", "Difícil", "Frenesi"};

    // Configuração de pontos customizada
    private final int[] OPCOES_PONTOS = {3, 5, 10, 15, 20};
    private int indiceOpcaoPontos = 1; // Padrão: 5 pontos
    private int pontosParaVencer = 5;

    // Índices de seleção dos menus
    private int opcaoMenuModo = 0;        // 0: Continuar, 1: 1P, 2: 2P, 3: Treino, 4: Opções
    private int opcaoMenuTipoJogo = 0;    // 0: Meta de Pontos, 1: Modo Infinito
    private int opcaoMenuOpcoes = 0;      // 0: Dif, 1: Cor J1, 2: Cor J2, 3: Cor Bola, 4: Tam Raquete, 5: Tam Bola, 6: Vel Raquete, 7: Voltar
    private int opcaoMenuPausa = 0;       // 0: Continuar, 1: Opções, 2: Resetar, 3: Voltar ao Menu
    
    // Customização de Cores
    private final Color[] CORES_DISPONIVEIS = {
        Color.WHITE, Color.RED, Color.GREEN, Color.BLUE, Color.YELLOW, Color.CYAN, Color.MAGENTA, Color.ORANGE
    };
    private final String[] NOMES_CORES = {
        "Branco", "Vermelho", "Verde", "Azul", "Amarelo", "Ciano", "Magenta", "Laranja"
    };

    private int idxCorJ1 = 0;
    private int idxCorJ2 = 0;
    private int idxCorBola = 0;

    private Color corJ1 = Color.WHITE;
    private Color corJ2 = Color.WHITE;
    private Color corBola = Color.WHITE;

    // Tamanhos Customizáveis
    private final int[] OPCOES_TAM_RAQUETE = {60, 100, 140};
    private final String[] NOMES_TAM_RAQUETE = {"Pequena (60px)", "Média (100px)", "Grande (140px)"};
    private int idxTamRaquete = 1;
    private int alturaRaquete = 100;

    private final int[] OPCOES_TAM_BOLA = {12, 20, 30};
    private final String[] NOMES_TAM_BOLA = {"Pequena (12px)", "Média (20px)", "Grande (30px)"};
    private int idxTamBola = 1;
    private int tamanhoBola = 20;

    // Velocidade da Raquete Customizável
    private final int[] OPCOES_VEL_RAQUETE = {4, 6, 9};
    private final String[] NOMES_VEL_RAQUETE = {"Lenta (4px)", "Média (6px)", "Rápida (9px)"};
    private int idxVelRaquete = 1;
    private int velocidadeRaquete = 6;

    private final int LARGURA_RAQUETE = 15;

    // Velocidades
    private double velocidadeBolaAtual = 4.0;
    private double velocidadeBolaBase = 4.0;
    private int velocidadeBot = 4;

    private boolean bolaEsperandoInicio = false;

    // Posições das raquetes
    private int j1Y = 250, j2Y = 250;

    // Posição e velocidade da bola
    private double bolaX = 400, bolaY = 300;
    private double bolaXDir = 4, bolaYDir = 4;

    // Pontuação
    private int pontosJ1 = 0, pontosJ2 = 0;
    private int rebatesTreinoAtual = 0;
    private int recordeTreino = 0;

    // Controles de movimento
    private boolean j1Cima, j1Baixo, j2Cima, j2Baixo;

    private final Random random = new Random();

    public operadoreslogicos() {
        this.setPreferredSize(new Dimension(LARGURA, ALTURA));
        this.setBackground(Color.BLACK);
        this.setFocusable(true);
        this.addKeyListener(this);

        Thread thread = new Thread(this);
        thread.start();
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);

        switch (estadoAtual) {
            case MENU_MODO:
                desenharMenuModo(g);
                break;
            case MENU_TIPO_JOGO:
                desenharMenuTipoJogo(g);
                break;
            case MENU_META_PONTOS:
                desenharMenuMetaPontos(g);
                break;
            case MENU_OPCOES:
                desenharMenuOpcoes(g);
                break;
            case JOGANDO:
                desenharJogo(g);
                break;
            case PAUSADO:
                desenharJogo(g);
                desenharMenuPausa(g);
                break;
            case FIM_DE_JOGO:
                desenharFimDeJogo(g);
                break;
        }
    }

    private void desenharMenuModo(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 48));
        g.drawString("PONG GAME", LARGURA / 2 - 150, 80);

        g.setFont(new Font("Arial", Font.PLAIN, 22));

        String op0 = (opcaoMenuModo == 0 ? "> " : "   ") + (temPartidaSalva ? "Continuar Partida Anterior" : "[Sem Partida Salva]");
        String op1 = (opcaoMenuModo == 1 ? "> " : "   ") + "1 Jogador (vs Bot)";
        String op2 = (opcaoMenuModo == 2 ? "> " : "   ") + "2 Jogadores";
        String op3 = (opcaoMenuModo == 3 ? "> " : "   ") + "Modo Treino (Solo)";
        String op4 = (opcaoMenuModo == 4 ? "> " : "   ") + "Opções do Jogo";

        g.setColor(!temPartidaSalva ? Color.GRAY : (opcaoMenuModo == 0 ? Color.YELLOW : Color.WHITE));
        g.drawString(op0, LARGURA / 2 - 160, 160);

        g.setColor(opcaoMenuModo == 1 ? Color.YELLOW : Color.WHITE);
        g.drawString(op1, LARGURA / 2 - 160, 210);

        g.setColor(opcaoMenuModo == 2 ? Color.YELLOW : Color.WHITE);
        g.drawString(op2, LARGURA / 2 - 160, 260);

        g.setColor(opcaoMenuModo == 3 ? Color.YELLOW : Color.WHITE);
        g.drawString(op3, LARGURA / 2 - 160, 310);

        g.setColor(opcaoMenuModo == 4 ? Color.YELLOW : Color.WHITE);
        g.drawString(op4, LARGURA / 2 - 160, 360);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 16));
        g.drawString("Use W/S ou SETAS para navegar | ESPAÇO para confirmar", LARGURA / 2 - 220, 440);
        g.drawString("Controles: J1 (W/S ou Setas no Bot/Treino) | J2 (Setas no 2P)", LARGURA / 2 - 240, 480);
    }

    private void desenharMenuOpcoes(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("OPÇÕES DO JOGO", LARGURA / 2 - 180, 70);

        g.setFont(new Font("Arial", Font.PLAIN, 18));

        String op0 = (opcaoMenuOpcoes == 0 ? "> " : "   ") + "Dificuldade: < " + NOMES_DIFICULDADE[dificuldadeAtual.ordinal()] + " >";
        String op1 = (opcaoMenuOpcoes == 1 ? "> " : "   ") + "Cor Raquete J1: < " + NOMES_CORES[idxCorJ1] + " >";
        String op2 = (opcaoMenuOpcoes == 2 ? "> " : "   ") + "Cor Raquete J2 / Parede: < " + NOMES_CORES[idxCorJ2] + " >";
        String op3 = (opcaoMenuOpcoes == 3 ? "> " : "   ") + "Cor da Bolinha: < " + NOMES_CORES[idxCorBola] + " >";
        String op4 = (opcaoMenuOpcoes == 4 ? "> " : "   ") + "Tamanho da Raquete: < " + NOMES_TAM_RAQUETE[idxTamRaquete] + " >";
        String op5 = (opcaoMenuOpcoes == 5 ? "> " : "   ") + "Tamanho da Bola: < " + NOMES_TAM_BOLA[idxTamBola] + " >";
        String op6 = (opcaoMenuOpcoes == 6 ? "> " : "   ") + "Velocidade Raquete: < " + NOMES_VEL_RAQUETE[idxVelRaquete] + " >";
        String op7 = (opcaoMenuOpcoes == 7 ? "> " : "   ") + "Voltar";

        g.setColor(opcaoMenuOpcoes == 0 ? Color.YELLOW : Color.WHITE);
        g.drawString(op0, LARGURA / 2 - 240, 130);

        g.setColor(opcaoMenuOpcoes == 1 ? Color.YELLOW : Color.WHITE);
        g.drawString(op1, LARGURA / 2 - 240, 175);
        g.setColor(CORES_DISPONIVEIS[idxCorJ1]);
        g.fillRect(LARGURA / 2 + 180, 160, 18, 18);

        g.setColor(opcaoMenuOpcoes == 2 ? Color.YELLOW : Color.WHITE);
        g.drawString(op2, LARGURA / 2 - 240, 220);
        g.setColor(CORES_DISPONIVEIS[idxCorJ2]);
        g.fillRect(LARGURA / 2 + 180, 205, 18, 18);

        g.setColor(opcaoMenuOpcoes == 3 ? Color.YELLOW : Color.WHITE);
        g.drawString(op3, LARGURA / 2 - 240, 265);
        g.setColor(CORES_DISPONIVEIS[idxCorBola]);
        g.fillOval(LARGURA / 2 + 180, 250, 18, 18);

        g.setColor(opcaoMenuOpcoes == 4 ? Color.YELLOW : Color.WHITE);
        g.drawString(op4, LARGURA / 2 - 240, 310);

        g.setColor(opcaoMenuOpcoes == 5 ? Color.YELLOW : Color.WHITE);
        g.drawString(op5, LARGURA / 2 - 240, 355);

        g.setColor(opcaoMenuOpcoes == 6 ? Color.YELLOW : Color.WHITE);
        g.drawString(op6, LARGURA / 2 - 240, 400);

        g.setColor(opcaoMenuOpcoes == 7 ? Color.YELLOW : Color.WHITE);
        g.drawString(op7, LARGURA / 2 - 240, 455);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 15));
        g.drawString("Navegar: CIMA/BAIXO (W/S) | Alterar: ESQUERDA/DIREITA (A/D)", LARGURA / 2 - 230, 520);
        g.drawString("Pressione ESPAÇO/ENTER ou ESC para confirmar/voltar", LARGURA / 2 - 210, 545);
    }

    private void desenharMenuTipoJogo(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("Tipo de Partida", LARGURA / 2 - 150, 130);

        g.setFont(new Font("Arial", Font.PLAIN, 24));

        String op0 = (opcaoMenuTipoJogo == 0 ? "> " : "   ") + "Partida por Pontos";
        String op1 = (opcaoMenuTipoJogo == 1 ? "> " : "   ") + "Modo Infinito";

        g.setColor(opcaoMenuTipoJogo == 0 ? Color.YELLOW : Color.WHITE);
        g.drawString(op0, LARGURA / 2 - 120, 260);

        g.setColor(opcaoMenuTipoJogo == 1 ? Color.YELLOW : Color.WHITE);
        g.drawString(op1, LARGURA / 2 - 120, 320);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 16));
        g.drawString("Use W/S ou SETAS para navegar | ESPAÇO para confirmar", LARGURA / 2 - 220, 450);
        g.drawString("Pressione ESC para voltar", LARGURA / 2 - 100, 490);
    }

    private void desenharMenuMetaPontos(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("Meta de Pontos", LARGURA / 2 - 150, 130);

        g.setFont(new Font("Arial", Font.PLAIN, 24));

        for (int i = 0; i < OPCOES_PONTOS.length; i++) {
            String texto = (indiceOpcaoPontos == i ? "> " : "   ") + OPCOES_PONTOS[i] + " Pontos";
            g.setColor(indiceOpcaoPontos == i ? Color.YELLOW : Color.WHITE);
            g.drawString(texto, LARGURA / 2 - 80, 220 + (i * 45));
        }

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 16));
        g.drawString("Use W/S ou SETAS para navegar | ESPAÇO para confirmar", LARGURA / 2 - 220, 470);
        g.drawString("Pressione ESC para voltar", LARGURA / 2 - 100, 500);
    }

    private void desenharJogo(Graphics g) {
        g.setColor(Color.WHITE);
        if (!modoTreino) {
            for (int i = 0; i < ALTURA; i += 30) {
                g.fillRect(LARGURA / 2 - 2, i, 4, 15);
            }
            g.setColor(corJ2);
            g.fillRect(LARGURA - 30 - LARGURA_RAQUETE, j2Y, LARGURA_RAQUETE, alturaRaquete);
        } else {
            g.setColor(corJ2);
            g.fillRect(LARGURA - 10, 0, 10, ALTURA);
        }

        g.setColor(corJ1);
        g.fillRect(30, j1Y, LARGURA_RAQUETE, alturaRaquete);

        g.setColor(corBola);
        g.fillOval((int) bolaX, (int) bolaY, tamanhoBola, tamanhoBola);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 20));
        if (modoTreino) {
            g.drawString("Rebates: " + rebatesTreinoAtual, 180, 50);
            g.drawString("Recorde: " + recordeTreino, 520, 50);
        } else {
            g.drawString("Jogador 1: " + pontosJ1, 180, 50);
            String nomeJ2 = modoBot ? "Bot: " : "Jogador 2: ";
            g.drawString(nomeJ2 + pontosJ2, 520, 50);
        }

        if (!modoTreino) {
            g.setFont(new Font("Arial", Font.ITALIC, 14));
            g.setColor(dificuldadeAtual == Dificuldade.FRENESI ? Color.ORANGE : Color.CYAN);
            String labelMeta = (tipoJogoAtual == TipoJogo.INFINITO) ? "Modo Infinito" : "Meta: " + pontosParaVencer + " pontos";
            if (dificuldadeAtual == Dificuldade.FRENESI) {
                labelMeta += " [FRENESI]";
            }
            g.drawString(labelMeta, LARGURA / 2 - 60, 25);
        }

        g.setFont(new Font("Arial", Font.PLAIN, 12));
        g.setColor(Color.GRAY);
        g.drawString("[ESC] Pausa", 10, 20);
        g.setColor(Color.WHITE);

        if (bolaEsperandoInicio) {
            g.setFont(new Font("Arial", Font.BOLD, 18));
            g.drawString("Pressione qualquer tecla de movimento para soltar a bola!", LARGURA / 2 - 250, 100);
        }
    }

    private void desenharMenuPausa(Graphics g) {
        g.setColor(new Color(0, 0, 0, 180));
        g.fillRect(0, 0, LARGURA, ALTURA);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));
        g.drawString("PAUSA", LARGURA / 2 - 70, 160);

        g.setFont(new Font("Arial", Font.PLAIN, 22));

        String op0 = (opcaoMenuPausa == 0 ? "> " : "   ") + "Continuar Jogo";
        String op1 = (opcaoMenuPausa == 1 ? "> " : "   ") + "Opções do Jogo";
        String op2 = (opcaoMenuPausa == 2 ? "> " : "   ") + "Resetar Pontos / Reiniciar";
        String op3 = (opcaoMenuPausa == 3 ? "> " : "   ") + "Voltar ao Menu Principal";

        g.setColor(opcaoMenuPausa == 0 ? Color.YELLOW : Color.WHITE);
        g.drawString(op0, LARGURA / 2 - 140, 240);

        g.setColor(opcaoMenuPausa == 1 ? Color.YELLOW : Color.WHITE);
        g.drawString(op1, LARGURA / 2 - 140, 290);

        g.setColor(opcaoMenuPausa == 2 ? Color.YELLOW : Color.WHITE);
        g.drawString(op2, LARGURA / 2 - 140, 340);

        g.setColor(opcaoMenuPausa == 3 ? Color.YELLOW : Color.WHITE);
        g.drawString(op3, LARGURA / 2 - 140, 390);

        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.ITALIC, 14));
        g.drawString("Use W/S ou SETAS | ESPAÇO/ENTER para selecionar | ESC para voltar", LARGURA / 2 - 220, 460);
    }

    private void desenharFimDeJogo(Graphics g) {
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 40));

        String vencedor = (pontosJ1 >= pontosParaVencer) ? "Jogador 1 Venceu!" : (modoBot ? "O Bot Venceu!" : "Jogador 2 Venceu!");
        g.drawString(vencedor, LARGURA / 2 - 180, 250);

        g.setFont(new Font("Arial", Font.PLAIN, 20));
        g.drawString("Pressione ESPAÇO ou ESC para voltar ao Menu", LARGURA / 2 - 220, 350);
    }

    private void aplicarDificuldade() {
        switch (dificuldadeAtual) {
            case FACIL:
                velocidadeBolaBase = 3.0;
                velocidadeBot = 2;
                break;
            case MEDIO:
                velocidadeBolaBase = 5.0;
                velocidadeBot = 4;
                break;
            case DIFICIL:
                velocidadeBolaBase = 7.0;
                velocidadeBot = 6;
                break;
            case FRENESI:
                velocidadeBolaBase = 3.0;
                velocidadeBot = 6;
                break;
        }

        // Atualiza a velocidade da bola atual mantendo a sua direção
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

        // Movimento do Jogador 1
        if (j1Cima && j1Y > 0) j1Y -= velocidadeRaquete;
        if (j1Baixo && j1Y < ALTURA - alturaRaquete) j1Y += velocidadeRaquete;

        // Movimento do Jogador 2 ou Bot
        if (!modoTreino) {
            if (modoBot) {
                if (bolaXDir > 0) {
                    int centroRaquete = j2Y + alturaRaquete / 2;
                    if (centroRaquete < bolaY + (tamanhoBola / 2) && j2Y < ALTURA - alturaRaquete) {
                        j2Y += velocidadeBot;
                    } else if (centroRaquete > bolaY + (tamanhoBola / 2) && j2Y > 0) {
                        j2Y -= velocidadeBot;
                    }
                }
            } else {
                if (j2Cima && j2Y > 0) j2Y -= velocidadeRaquete;
                if (j2Baixo && j2Y < ALTURA - alturaRaquete) j2Y += velocidadeRaquete;
            }
        }

        // Movimento da bola
        if (!bolaEsperandoInicio) {
            bolaX += bolaXDir;
            bolaY += bolaYDir;

            // Colisão com topo e base
            if (bolaY <= 0 || bolaY >= ALTURA - tamanhoBola) {
                bolaYDir = -bolaYDir;
            }

            // Colisão com Raquete 1
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

            // Colisão com o lado direito
            if (modoTreino) {
                if (bolaX + tamanhoBola >= LARGURA - 10) {
                    bolaXDir = -Math.abs(bolaXDir);
                    bolaX = LARGURA - 10 - tamanhoBola - 1;
                    aumentarVelocidadeFrenesi();
                }
            } else {
                if (bolaX + tamanhoBola >= LARGURA - 30 - LARGURA_RAQUETE && bolaX + tamanhoBola <= LARGURA - 30) {
                    if (bolaY + tamanhoBola >= j2Y && bolaY <= j2Y + alturaRaquete) {
                        bolaXDir = -Math.abs(bolaXDir);
                        bolaX = LARGURA - 30 - LARGURA_RAQUETE - tamanhoBola - 1;
                        aumentarVelocidadeFrenesi();
                    }
                }
            }

            // Pontuação e reset ao sair da tela
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
            angulo = (random.nextDouble() * Math.PI / 2) - (Math.PI / 4);
        } else {
            boolean paraDireita = random.nextBoolean();
            double variacao = (random.nextDouble() * Math.PI / 2) - (Math.PI / 4);
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
        j1Y = (ALTURA - alturaRaquete) / 2;
        j2Y = (ALTURA - alturaRaquete) / 2;
        j1Cima = j1Baixo = j2Cima = j2Baixo = false;
        aplicarDificuldade();
        reiniciarBola();
        if (modoBot) {
            bolaEsperandoInicio = false;
        }
        temPartidaSalva = true;
    }

    private void alterarOpcaoOpcoes(int direcao) {
        switch (opcaoMenuOpcoes) {
            case 0: // Dificuldade
                int totalDif = Dificuldade.values().length;
                int novaDif = (dificuldadeAtual.ordinal() + direcao + totalDif) % totalDif;
                dificuldadeAtual = Dificuldade.values()[novaDif];
                aplicarDificuldade(); // Aplica a nova velocidade na hora
                break;
            case 1: // Cor J1
                idxCorJ1 = (idxCorJ1 + direcao + CORES_DISPONIVEIS.length) % CORES_DISPONIVEIS.length;
                corJ1 = CORES_DISPONIVEIS[idxCorJ1];
                break;
            case 2: // Cor J2
                idxCorJ2 = (idxCorJ2 + direcao + CORES_DISPONIVEIS.length) % CORES_DISPONIVEIS.length;
                corJ2 = CORES_DISPONIVEIS[idxCorJ2];
                break;
            case 3: // Cor Bola
                idxCorBola = (idxCorBola + direcao + CORES_DISPONIVEIS.length) % CORES_DISPONIVEIS.length;
                corBola = CORES_DISPONIVEIS[idxCorBola];
                break;
            case 4: // Tamanho Raquete
                idxTamRaquete = (idxTamRaquete + direcao + OPCOES_TAM_RAQUETE.length) % OPCOES_TAM_RAQUETE.length;
                alturaRaquete = OPCOES_TAM_RAQUETE[idxTamRaquete];
                break;
            case 5: // Tamanho Bola
                idxTamBola = (idxTamBola + direcao + OPCOES_TAM_BOLA.length) % OPCOES_TAM_BOLA.length;
                tamanhoBola = OPCOES_TAM_BOLA[idxTamBola];
                break;
            case 6: // Velocidade Raquete
                idxVelRaquete = (idxVelRaquete + direcao + OPCOES_VEL_RAQUETE.length) % OPCOES_VEL_RAQUETE.length;
                velocidadeRaquete = OPCOES_VEL_RAQUETE[idxVelRaquete];
                break;
        }
    }

    @Override
    public void run() {
        while (true) {
            atualizar();
            repaint();
            try {
                Thread.sleep(6);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int codigo = e.getKeyCode();

        if (estadoAtual == EstadoJogo.MENU_MODO) {
            if (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_UP) {
                opcaoMenuModo = (opcaoMenuModo - 1 + 5) % 5;
            } else if (codigo == KeyEvent.VK_S || codigo == KeyEvent.VK_DOWN) {
                opcaoMenuModo = (opcaoMenuModo + 1) % 5;
            } else if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER) {
                if (opcaoMenuModo == 0) {
                    if (temPartidaSalva) {
                        estadoAtual = EstadoJogo.JOGANDO;
                    }
                } else if (opcaoMenuModo == 4) {
                    estadoAnteriorOpcoes = EstadoJogo.MENU_MODO;
                    estadoAtual = EstadoJogo.MENU_OPCOES;
                } else {
                    modoBot = (opcaoMenuModo == 1);
                    modoTreino = (opcaoMenuModo == 3);

                    if (modoTreino) {
                        resetarPartida(true);
                        estadoAtual = EstadoJogo.JOGANDO;
                    } else {
                        estadoAtual = EstadoJogo.MENU_TIPO_JOGO;
                    }
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_OPCOES) {
            if (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_UP) {
                opcaoMenuOpcoes = (opcaoMenuOpcoes - 1 + 8) % 8;
            } else if (codigo == KeyEvent.VK_S || codigo == KeyEvent.VK_DOWN) {
                opcaoMenuOpcoes = (opcaoMenuOpcoes + 1) % 8;
            } else if (codigo == KeyEvent.VK_D || codigo == KeyEvent.VK_RIGHT) {
                alterarOpcaoOpcoes(1);
            } else if (codigo == KeyEvent.VK_A || codigo == KeyEvent.VK_LEFT) {
                alterarOpcaoOpcoes(-1);
            } else if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER || codigo == KeyEvent.VK_ESCAPE) {
                if (opcaoMenuOpcoes == 7 || codigo == KeyEvent.VK_ESCAPE || codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER) {
                    estadoAtual = estadoAnteriorOpcoes;
                }
            }
        } else if (estadoAtual == EstadoJogo.MENU_TIPO_JOGO) {
            if (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_UP) {
                opcaoMenuTipoJogo = (opcaoMenuTipoJogo - 1 + 2) % 2;
            } else if (codigo == KeyEvent.VK_S || codigo == KeyEvent.VK_DOWN) {
                opcaoMenuTipoJogo = (opcaoMenuTipoJogo + 1) % 2;
            } else if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER) {
                tipoJogoAtual = (opcaoMenuTipoJogo == 0) ? TipoJogo.PONTOS_DEFINIDOS : TipoJogo.INFINITO;
                if (tipoJogoAtual == TipoJogo.PONTOS_DEFINIDOS) {
                    estadoAtual = EstadoJogo.MENU_META_PONTOS;
                } else {
                    resetarPartida(true);
                    estadoAtual = EstadoJogo.JOGANDO;
                }
            } else if (codigo == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_MODO;
            }
        } else if (estadoAtual == EstadoJogo.MENU_META_PONTOS) {
            if (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_UP) {
                indiceOpcaoPontos = (indiceOpcaoPontos - 1 + OPCOES_PONTOS.length) % OPCOES_PONTOS.length;
            } else if (codigo == KeyEvent.VK_S || codigo == KeyEvent.VK_DOWN) {
                indiceOpcaoPontos = (indiceOpcaoPontos + 1) % OPCOES_PONTOS.length;
            } else if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER) {
                pontosParaVencer = OPCOES_PONTOS[indiceOpcaoPontos];
                resetarPartida(true);
                estadoAtual = EstadoJogo.JOGANDO;
            } else if (codigo == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_TIPO_JOGO;
            }
        } else if (estadoAtual == EstadoJogo.JOGANDO) {
            if (codigo == KeyEvent.VK_ESCAPE) {
                opcaoMenuPausa = 0;
                estadoAtual = EstadoJogo.PAUSADO;
                return;
            }

            if (bolaEsperandoInicio && (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_S || 
                                        codigo == KeyEvent.VK_UP || codigo == KeyEvent.VK_DOWN)) {
                bolaEsperandoInicio = false;
            }

            if (codigo == KeyEvent.VK_W || ((modoBot || modoTreino) && codigo == KeyEvent.VK_UP)) j1Cima = true;
            if (codigo == KeyEvent.VK_S || ((modoBot || modoTreino) && codigo == KeyEvent.VK_DOWN)) j1Baixo = true;

            if (!modoBot && !modoTreino) {
                if (codigo == KeyEvent.VK_UP) j2Cima = true;
                if (codigo == KeyEvent.VK_DOWN) j2Baixo = true;
            }
        } else if (estadoAtual == EstadoJogo.PAUSADO) {
            if (codigo == KeyEvent.VK_W || codigo == KeyEvent.VK_UP) {
                opcaoMenuPausa = (opcaoMenuPausa - 1 + 4) % 4;
            } else if (codigo == KeyEvent.VK_S || codigo == KeyEvent.VK_DOWN) {
                opcaoMenuPausa = (opcaoMenuPausa + 1) % 4;
            } else if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER) {
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
            } else if (codigo == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.JOGANDO;
            }
        } else if (estadoAtual == EstadoJogo.FIM_DE_JOGO) {
            if (codigo == KeyEvent.VK_SPACE || codigo == KeyEvent.VK_ENTER || codigo == KeyEvent.VK_ESCAPE) {
                estadoAtual = EstadoJogo.MENU_MODO;
            }
        }
    }

    @Override
    public void keyReleased(KeyEvent e) {
        int codigo = e.getKeyCode();

        if (codigo == KeyEvent.VK_W || ((modoBot || modoTreino) && codigo == KeyEvent.VK_UP)) j1Cima = false;
        if (codigo == KeyEvent.VK_S || ((modoBot || modoTreino) && codigo == KeyEvent.VK_DOWN)) j1Baixo = false;

        if (!modoBot && !modoTreino) {
            if (codigo == KeyEvent.VK_UP) j2Cima = false;
            if (codigo == KeyEvent.VK_DOWN) j2Baixo = false;
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {}

    public static void main(String[] args) {
        JFrame janela = new JFrame("Jogo Pong - Java");
        operadoreslogicos jogo = new operadoreslogicos();
        janela.add(jogo);
        janela.pack();
        janela.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        janela.setLocationRelativeTo(null);
        janela.setVisible(true);
    }
}
