# chat-app-in-java
chat application written in java
package zappa;

import java.awt.BorderLayout;
import java.awt.Dimension;
import java.awt.EventQueue;

import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.JButton;
import javax.swing.JComboBox;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;
import java.awt.Color;
import java.awt.Font;
import javax.swing.JMenuBar;
import javax.swing.JMenuItem;
import javax.swing.JMenu;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

public class ClientGUI extends JFrame {

	private static final long serialVersionUID = 1L;
	private JPanel contentPane;
	private JScrollPane scrollPane;
	private JTextArea textArea;
	private JPanel panel;
	
	
	private JPanel panel_1;
	private JLabel lblUnosPoruke;
	private JTextField textField;
	private JButton btnPosaljiPoruku;
	private JMenuBar menuBar;
	private JMenu mnEdit;
	private JMenu mnHelp;
	private JMenuItem menuItem1;

	/**
	 * Launch the application.
	 */

	/**
	 * Create the frame.
	 */
	public ClientGUI() {
		setTitle("Client GUI");
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setBounds(100, 100, 450, 300);
		setJMenuBar(getMenuBar_1());
		contentPane = new JPanel();
		contentPane.setBackground(new Color(255, 255, 204));
		contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));
		setContentPane(contentPane);
		contentPane.setLayout(new BorderLayout(0, 0));
		contentPane.add(getScrollPane(), BorderLayout.CENTER);
		contentPane.add(getPanel(), BorderLayout.SOUTH);
		contentPane.add(getPanel_1(), BorderLayout.SOUTH);
	}

	private JScrollPane getScrollPane() {
		if (scrollPane == null) {
			scrollPane = new JScrollPane();
			scrollPane.setViewportView(getTextArea1());
		}
		return scrollPane;
	}
	public JTextArea getTextArea1() {
		if (textArea == null) {
			textArea = new JTextArea();
			textArea.setLineWrap(true);
		}
		return textArea;
	}
	private JPanel getPanel() {
		if (panel == null) {
			panel = new JPanel();
			
		
		}
		return panel;
	}
	
	private JPanel getPanel_1() {
		if (panel_1 == null) {
			panel_1 = new JPanel();
			panel_1.setBackground(new Color(153, 204, 204));
			panel_1.add(getLblUnosPoruke());
			panel_1.add(getTextField());
			panel_1.add(getBtnPosaljiPoruku());
		}
		return panel_1;
	}
	private JLabel getLblUnosPoruke() {
		if (lblUnosPoruke == null) {
			lblUnosPoruke = new JLabel("Your message : ");
		}
		return lblUnosPoruke;
	}
	private JTextField getTextField() {
		if (textField == null) {
			textField = new JTextField();
			textField.setColumns(17);
		}
		return textField;
	}
	private JButton getBtnPosaljiPoruku() {
		if (btnPosaljiPoruku == null) {
			btnPosaljiPoruku = new JButton("Send");
			btnPosaljiPoruku.setForeground(new Color(0, 51, 0));
			btnPosaljiPoruku.addActionListener(new ActionListener() {
				public void actionPerformed(ActionEvent arg0) {
					Client.izlazniTokKaServeru.println(getMessageToServer());
					textField.setText(null);
				}
			});
			btnPosaljiPoruku.setFont(new Font("Times New Roman", Font.BOLD, 13));
			btnPosaljiPoruku.setBackground(new Color(153, 153, 153));
		}
		return btnPosaljiPoruku;
	}
	public void myAppend(String lajna) {
		textArea.append(lajna + "\n");
	}
	public String getMessageToServer() {
		return textField.getText();
	}
	private JMenuBar getMenuBar_1() {
		if (menuBar == null) {
			menuBar = new JMenuBar();
			menuBar.add(getMnEdit());
			menuBar.add(getMnHelp());
		}
		return menuBar;
	}
	private JMenuItem getMenuItem1() {
		if(menuItem1 == null) {
			menuItem1 = new JMenuItem("Clear");
			menuItem1.addActionListener(new ActionListener() {
				
				@Override
				public void actionPerformed(ActionEvent e) {
					textArea.setText("");
					
				}
			});
			
		}
		return menuItem1;
	}
	private JMenu getMnEdit() {
		if (mnEdit == null) {
			mnEdit = new JMenu("Edit");
			mnEdit.add(getMenuItem1());
			
		}
		return mnEdit;
	}
	private JMenu getMnHelp() {
		if (mnHelp == null) {
			mnHelp = new JMenu("Help");
		}
		return mnHelp;
	}
	/*public static void main(String[] args) {
		new ClientGUI().setVisible(true);
	}*/
}


