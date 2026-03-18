# Enviar apenas o DevSquad ao GitHub

Este diretório (`devsquad-standalone`) contém **somente** o DevSquad, sem Opensquad nem outros squads.

## Passos

1. **Crie um repositório vazio no GitHub**  
   Ex.: `https://github.com/SEU_USUARIO/devsquad`

2. **Abra o terminal nesta pasta** (`devsquad-standalone`):

   ```powershell
   cd c:\dev\squad\devsquad-standalone
   ```

3. **Inicialize o Git e faça o primeiro commit:**

   ```powershell
   git init
   git add .
   git commit -m "chore: DevSquad standalone - code review com Lucas Tech Lead e equipe"
   git branch -M main
   ```

4. **Conecte ao GitHub e envie:**

   ```powershell
   git remote add origin https://github.com/SEU_USUARIO/devsquad.git
   git push -u origin main
   ```

   (Troque `SEU_USUARIO/devsquad` pela URL do seu repositório.)

Pronto. O repositório no GitHub terá só o DevSquad, isolado do resto do projeto (Opensquad, squads, etc.).
