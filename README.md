
# Passo a Passo de Substituição do Código no `main.ts`, Implementação do Prisma, Adequação do Swagger para `UtilityService` e Configuração do PostgreSQL com Render.com

## 1. Localize o Arquivo `main.ts`

Navegue até o arquivo `main.ts` dentro da pasta `src` do seu projeto NestJS.

## 2. Substitua o Código Existente

Substitua o conteúdo do arquivo `main.ts` pelo código abaixo:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Configuração básica do Swagger
  const config = new DocumentBuilder()
    .setTitle('API Site institucional')
    .setDescription('API para gerenciamento Pagina institucional')
    .setVersion('1.0')
    .addTag('Utility')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  const port = 3001;
  await app.listen(port);

  // Imprimir link clicável no terminal
  console.log(`Application is running on: http://localhost:${port}/api`);
}
bootstrap();
```

## 3. Conectar o Prisma ao PostgreSQL com Render.com

### 3.1 Criação de Conta no Render.com

1. Acesse o site do Render: [render.com](https://render.com).
2. Clique no botão **"Get Started"** no canto superior direito.
3. Na página de login, clique em **"Continue with GitHub"**.
   - O Render solicitará acesso ao seu GitHub. Autorize o acesso.
4. Complete as informações adicionais, se necessário, para finalizar a criação da conta.

### 3.2 Criação de um Banco de Dados PostgreSQL Gratuito

1. No painel principal, clique no botão **"New"** e selecione **"Database"**.
2. Escolha **"PostgreSQL"** como o tipo de banco de dados.
3. Preencha os detalhes:
   - **Name**: Escolha um nome para o banco.
   - **Database Region**: Selecione a região mais próxima para melhor performance.
   - **Instance Type**: Selecione **"Free"** para usar a versão gratuita.
4. Clique em **Create Database**.

### 3.3 Obtenção do Link de Conexão (Connection String)

1. Após a criação, você será redirecionado para a página de detalhes do banco de dados.
2. Na seção **Connection** (Conexão), você verá a **Connection String** no formato:
   ```plaintext
   postgres://USERNAME:PASSWORD@HOST:PORT/DATABASE
   ```
3. Copie a **Connection String** para usar no arquivo `.env`.

### 3.4 Conectar o Prisma ao PostgreSQL

No arquivo `.env`, adicione a variável `DATABASE_URL` com a Connection String obtida no Render:

```env
DATABASE_URL="postgres://USERNAME:PASSWORD@HOST:PORT/DATABASE"
```

## 4. Adicionar e Configurar o Prisma

### 4.1 Instale o Prisma e o Cliente Prisma (com Yarn)

No diretório raiz do seu projeto, execute o comando para instalar o Prisma e o cliente Prisma:

```bash
yarn add -D prisma
yarn add @prisma/client
```

### 4.2 Inicialize o Prisma

Inicialize o Prisma no projeto com o seguinte comando:

```bash
yarn prisma init
```

Isso criará uma pasta `prisma` e um arquivo `schema.prisma`. O arquivo `schema.prisma` é onde você definirá o modelo de dados da sua aplicação.

### 4.3 Configure o `schema.prisma` com UUID

Abra o arquivo `schema.prisma` gerado e adicione o seguinte conteúdo, adaptado para inglês, e configure o `id` como `UUID`:

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Utility {
  id          String   @id @default(uuid()) @db.Uuid
  title       String
  description String
  image       String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

### 4.4 Execute as Migrações

Após configurar os modelos, você pode criar e aplicar migrações para sincronizar o banco de dados com os modelos do Prisma:

```bash
yarn prisma migrate dev --name add-utility
```

### 4.5 Gerar o Cliente Prisma

Para gerar o cliente Prisma, execute o comando abaixo:

```bash
yarn prisma generate
```
### 4.6 Criar o PrismaService

Crie o arquivo `prisma.service.ts`

Dentro da pasta `src/prisma/`, crie o arquivo `prisma.service.ts` com o seguinte conteúdo:

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

### 4.7 Crie o arquivo `prisma.module.ts`

No mesmo diretório, crie o arquivo `prisma.module.ts` com o seguinte conteúdo:

```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

### 4.8 Importe o PrismaModule no AppModule

No arquivo `app.module.ts`, importe o `PrismaModule`:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UtilityModule } from './utility/utility.module';
import { PrismaModule } from './prisma/prisma.module';

@Module({
  imports: [UtilityModule, PrismaModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

## 5. Criar o DTO para `Utility`

### 5.1 Crie os arquivos `create-utility.dto.ts` e `update-utility.dto.ts`

Dentro da pasta `src/utility/dto/`, crie os arquivos `create-utility.dto.ts` e `update-utility.dto.ts` com o seguinte conteúdo:

**`create-utility.dto.ts`**

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class CreateUtilityDto {
  @ApiProperty({
    description: 'Title of the utility',
    example: 'My Utility',
  })
  title: string;

  @ApiProperty({
    description: 'Description of the utility',
    example: 'This utility is used to manage something specific.',
  })
  description: string;

  @ApiProperty({
    description: 'Image related to the utility',
    example: 'https://image-link.com/image.jpg',
  })
  image: string;
}
```

**`update-utility.dto.ts`**

```typescript
import { PartialType } from '@nestjs/swagger';
import { CreateUtilityDto } from './create-utility.dto';

export class UpdateUtilityDto extends PartialType(CreateUtilityDto) {}
```

## 6. Adequar o `UtilityService` e `UtilityController`

### 6.1 Atualize o Serviço `UtilityService`

No arquivo `utility.service.ts`, altere o código para utilizar o Prisma e criar, atualizar, buscar e remover os dados de `Utility` no banco de dados:

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from 'src/prisma/prisma.service';
import { CreateUtilityDto } from './dto/create-utility.dto';
import { UpdateUtilityDto } from './dto/update-utility.dto';

@Injectable()
export class UtilityService {
  constructor(private prisma: PrismaService) {}

  async create(createUtilityDto: CreateUtilityDto) {
    return this.prisma.utility.create({
      data: createUtilityDto,
    });
  }

  async findAll() {
    return this.prisma.utility.findMany();
  }

  async findOne(id: string) {
    return this.prisma.utility.findUnique({
      where: { id },
    });
  }

  async update(id: string, updateUtilityDto: UpdateUtilityDto) {
    return this.prisma.utility.update({
      where: { id },
      data: updateUtilityDto,
    });
  }

  async remove(id: string) {
    return this.prisma.utility.delete({
      where: { id },
    });
  }
}
```

### 6.2 Atualize o Controlador `UtilityController`

No arquivo `utility.controller.ts`, o controlador permanece o mesmo, mas agora ele está utilizando o Prisma para lidar com os dados de `Utility` no banco de dados:

```typescript
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common';
import { UtilityService } from './utility.service';
import { CreateUtilityDto } from './dto/create-utility.dto';
import { UpdateUtilityDto } from './dto/update-utility.dto';
import { ApiTags } from '@nestjs/swagger';

@ApiTags('Utility')
@Controller('utility')
export class UtilityController {
  constructor(private readonly utilityService: UtilityService) {}

  @Post()
  create(@Body() createUtilityDto: CreateUtilityDto) {
    return this.utilityService.create(createUtilityDto);
  }

  @Get()
  findAll() {
    return this.utilityService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.utilityService.findOne(id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUtilityDto: UpdateUtilityDto) {
    return this.utilityService.update(id, updateUtilityDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.utilityService.remove(id);
  }
}
```

## 7. Salve as Alterações

Após substituir o código, salve todas as alterações.

## 8. Execute o Projeto

Execute o projeto usando o comando abaixo para iniciar o servidor NestJS:

```bash
yarn start:dev
```

## 9. Verifique a Documentação da API

Acesse a documentação do Swagger gerada automaticamente no seu navegador através do link:

```
http://localhost:3001/api
```

Agora você poderá visualizar e interagir com a documentação da API gerada pelo Swagger, incluindo as operações de `Utility` para criar, atualizar, buscar e remover registros.
