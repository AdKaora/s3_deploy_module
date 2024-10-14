# Deploy to S3 and Invalidate CloudFront

Questo workflow GitHub Actions consente di fare upload su S3 e, opzionalmente, di invalidare la cache di Amazon CloudFront.

## Prerequisiti

Prima di utilizzare questo workflow, assicurati di avere:

- Un bucket S3 configurato per ospitare il tuo sito o applicazione.
- Configurato i segreti nel tuo repository GitHub per gestire in modo sicuro le credenziali AWS.

## Input

Il workflow accetta i seguenti input:

| Input               | Descrizione                                 | Richiesto | Tipo    | Valore Predefinito |
| ------------------- | ------------------------------------------- | --------- | ------- | ------------------ |
| `AWS_REGION`        | Regione AWS                                 | Sì        | string  | -                  |
| `S3_BUCKET`         | Nome del bucket S3                           | Sì        | string  | -                  |
| `SOURCE_DIR`        | Directory da caricare su S3                 | Sì        | string  | -                  |
| `INVALIDATE_CLOUDFRONT` | Se invalidare la cache di CloudFront         | No        | boolean | `false`            |
| `DISTRIBUTION_ID`   | ID della distribuzione CloudFront            | No        | string  | -                  |

## Secrets

Il workflow richiede i seguenti secrets:

| Secret         | Descrizione                           | Richiesto |
| -------------- | ------------------------------------- | --------- |
| `AWS_ROLE_ARN` | ARN del ruolo AWS da assumere          | Sì        |


## Jobs

### deploy

Esegue il processo di deploy seguendo questi passaggi:

1. **Checkout del codice**: Clona il repository nel runner GitHub.
2. **Scarica gli artifact**: Scarica gli artifact di build precedentemente generati e li posiziona nella directory specificata.
3. **Configura AWS CLI**: Configura le credenziali AWS utilizzando il ruolo specificato.
4. **Sincronizza con S3**: Sincronizza i file dalla directory di origine al bucket S3, eliminando i file obsoleti.
5. **Invalidare CloudFront**: (Opzionale) Crea un'invalidazione della cache per la distribuzione CloudFront specificata.

## Esempio di Utilizzo

Per utilizzare questo workflow riutilizzabile nel tuo repository, devi richiamarlo da un altro workflow. Di seguito sono riportati i passaggi dettagliati su come configurarlo. Crea un file di workflow nel tuo repository (ad esempio .github/workflows/deploy.yml) con il seguente contenuto:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - master
      
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    needs: build
    uses: AdKaora/s3_deploy_module/.github/workflows/deploy_s3.yml@master
    with:
      AWS_REGION: ${{ vars.AWS_REGION }}
      S3_BUCKET: ${{ vars.S3_BUCKET }}
      SOURCE_DIR: out
      INVALIDATE_CLOUDFRONT: true
      DISTRIBUTION_ID: ${{ vars.DISTRIBUTION_ID }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}

```

