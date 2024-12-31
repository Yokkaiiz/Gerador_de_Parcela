import os
import sys
import webbrowser
from datetime import datetime
from flask import Flask, render_template, request, send_file, redirect
from fpdf import FPDF

# Certifique-se de que a pasta 'temp_pdfs' exista
if not os.path.exists('temp_pdfs'):
    os.makedirs('temp_pdfs')

app = Flask(__name__)

def resource_path(relative_path):
    """Obtenha o caminho absoluto para os recursos, mesmo em um executável."""
    try:
        # Caminho para o executável
        base_path = sys._MEIPASS
    except AttributeError:
        # Caminho para o script Python
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/gerar-pdf', methods=['POST'])
def gerar_pdf():
    try:
        # Coleta os dados do formulário
        capa = request.files['capa']
        nome_cliente = request.form.get('nome_cliente', 'N/A')
        parcelas = int(request.form['parcelas'])
        valor_total = float(request.form['valor'])
        vencimento_str = request.form['vencimento']
        vencimento = datetime.strptime(vencimento_str, '%Y-%m-%d')

        dia_fixo = vencimento.day
        valor_parcela = round(valor_total / parcelas, 2)

        # Criar diretório temporário para armazenar os arquivos
        temp_dir = "temp_pdfs"
        os.makedirs(temp_dir, exist_ok=True)

        # Caminho para arquivo temporário da capa
        temp_capa_path = os.path.join(temp_dir, capa.filename)
        capa.save(temp_capa_path)

        # Caminho para o QR Code que está na pasta static
        qr_code_path = resource_path('static/images/qrcode_igreja.png')

        # Criar o PDF com FPDF
        pdf = FPDF()
        pdf.set_auto_page_break(auto=True, margin=20)

        # Adicionar a capa (página única)
        pdf.add_page()
        if os.path.exists(temp_capa_path):
            pdf.image(temp_capa_path, x=10, y=8, w=190)
        os.remove(temp_capa_path)

        # Adiciona uma nova página para as parcelas
        pdf.add_page()

        # Gerar as parcelas (4 parcelas por página)
        for i in range(1, parcelas + 1):
            mes_vencimento = vencimento.month + (i - 1)
            ano_vencimento = vencimento.year + (mes_vencimento - 1) // 12
            mes_vencimento = (mes_vencimento - 1) % 12 + 1
            data_vencimento = datetime(ano_vencimento, mes_vencimento, dia_fixo)

            if (i - 1) % 4 == 0 and i != 1:
                pdf.add_page()

            pdf.set_font("Arial", style='B', size=17)
            pdf.cell(0, 15, "CONTROLE DE PAGAMENTO", ln=True, align='C')
            pdf.set_font("Arial", size=12)
            pdf.cell(0, 7, f"Nome: {nome_cliente}", ln=True)
            pdf.cell(0, 7, f"Quantidade de Parcela {i}/{parcelas}", ln=True)
            pdf.cell(0, 7, f"Valor das Parcelas: R$ {valor_parcela:,.2f}", ln=True)
            pdf.cell(0, 7, f"Data de Vencimento: {data_vencimento.strftime('%d/%m/%Y')}", ln=True)
            pdf.cell(0, 7, f"Data do Pagamento: ____/____/____", ln=True)
            pdf.cell(0, 7, f"Assinatura: _____________________", ln=True)

            # Linha pontilhada vertical
            pdf.set_line_width(0.1)
            x1 = 105
            y1 = int(pdf.get_y()) - 47
            y2 = int(pdf.get_y()) + 3
            for y in range(y1, y2, 2):
                pdf.line(x1, int(y), x1, int(y + 1))

            # Linha pontilhada horizontal
            pdf.set_line_width(0.1)
            y1_horizontal = int(pdf.get_y()) + 5
            x1_horizontal = 10
            x2_horizontal = 200
            for x in range(x1_horizontal, x2_horizontal, 2):
                pdf.line(x, y1_horizontal, x + 1, y1_horizontal)

            # Adicionar o QR Code no lado direito
            if os.path.exists(qr_code_path):
                qr_code_x = 110
                qr_code_y = pdf.get_y() - 40
                pdf.image(qr_code_path, x=qr_code_x, y=qr_code_y, w=33)

                text_x = qr_code_x + 35
                text_y = qr_code_y + 10
                pdf.set_font("Arial", size=10)
                pdf.text(text_x, text_y, "QR Code Pix - Pr. Junior")

            if i % 4 != 0:
                pdf.ln(10)

        pdf_filename = os.path.join(temp_dir, "carne_pagamento.pdf")
        pdf.output(pdf_filename)

        return send_file(pdf_filename, as_attachment=True, download_name="carne_pagamento.pdf", mimetype="application/pdf")

    except Exception:
        return redirect('/')  # Redireciona para a página inicial em caso de erro

if __name__ == '__main__':
    # Abre o navegador automaticamente
    url = "http://127.0.0.1:5000"
    webbrowser.open(url)
    app.run(debug=True)
