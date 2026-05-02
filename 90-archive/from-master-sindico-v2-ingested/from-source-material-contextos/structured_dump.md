\n\n===== FILE: ./package.json =====
{
	"name": "api",
	"type": "module",
	"private": true,
	"main": "dist/src/server.js",
	"directories": {
		"test": "test"
	},
	"scripts": {
		"build": "swc src -d dist --copy-files",
		"dev": "tsx watch --env-file=./.env src/server.ts",
		"typecheck": "tsc --noEmit",
		"start": "bun --env-file=./.env dist/src/server.js"
	},
	"devDependencies": {
		"@swc/cli": "^0.7.10",
		"@swc/core": "^1.15.11",
		"@types/bun": "^1.3.8",
		"@types/node": "^25.0.10",
		"@types/nodemailer": "^7.0.9",
		"@types/pg": "^8.16.0",
		"drizzle-kit": "^1.0.0-beta.12-a5629fb",
		"pino-pretty": "^13.1.3",
		"tsx": "^4.21.0",
		"typescript": "^5.9.3"
	},
	"dependencies": {
		"@fastify/awilix": "^8.2.0",
		"@fastify/cookie": "^11.0.2",
		"@fastify/cors": "^11.2.0",
		"@fastify/helmet": "^13.0.2",
		"@fastify/middie": "^9.1.0",
		"@fastify/multipart": "^9.4.0",
		"@fastify/rate-limit": "^10.3.0",
		"@fastify/redis": "^7.1.0",
		"@fastify/schedule": "^6.0.0",
		"@fastify/swagger": "^9.6.1",
		"@scalar/fastify-api-reference": "^1.44.0",
		"argon2": "^0.44.0",
		"awilix": "^12.0.5",
		"dotenv": "^17.2.3",
		"drizzle-orm": "^1.0.0-beta.12-a5629fb",
		"drizzle-zod": "^0.8.3",
		"fastify": "^5.7.2",
		"fastify-plugin": "^5.1.0",
		"fastify-zod-openapi": "^5.5.0",
		"jose": "^6.1.3",
		"mastersindico-schemas": "workspace:*",
		"nanoid": "^5.1.6",
		"nodemailer": "^7.0.13",
		"pg": "^8.17.2",
		"resend": "^6.9.1",
		"stripe": "^20.3.1",
		"toad-scheduler": "^3.1.0",
		"ua-parser-js": "^2.0.8",
		"uuid": "^13.0.0",
		"zod": "^4.3.6"
	}
}\n\n===== FILE: ./tsconfig.json =====
{
	"compilerOptions": {
		// Environment setup & latest features
		"lib": ["ESNext"],
		"target": "ESNext",
		"module": "Preserve",
		"moduleDetection": "force",
		"jsx": "react-jsx",
		"allowJs": true,

		// Bundler mode
		"moduleResolution": "bundler",
		"allowImportingTsExtensions": true,
		"verbatimModuleSyntax": true,
		"noEmit": true,

		// Best practices
		"strict": true,
		"skipLibCheck": true,
		"noFallthroughCasesInSwitch": true,
		"noUncheckedIndexedAccess": true,
		"noImplicitOverride": true,

		// Some stricter flags (disabled by default)
		"noUnusedLocals": false,
		"noUnusedParameters": false,
		"noPropertyAccessFromIndexSignature": false
	},
	"include": ["./src/**/*", "./src/shared/types/fastify.d.ts"]
}
\n\n===== FILE: ./src/app.ts =====
import Fastify from "fastify";
import type { ZodTypeProvider } from "fastify-type-provider-zod";
import { errorHandler } from "./infrastructure/handlers/errors.handler";
import { AppModule } from "./modules/app.module";
import { env } from "./shared/config";
import awilixPlugin from "./shared/plugins/awilix.plugin";
import cookiePlugin from "./shared/plugins/cookie.plugin";
import corsPlugin from "./shared/plugins/cors.plugin";
import helmetPlugin from "./shared/plugins/helmet.plugin";
import loggerPlugin from "./shared/plugins/logger.plugin";
import middlePlugin from "./shared/plugins/middle.plugin";
import multipartPlugin from "./shared/plugins/multipart.plugin";
import rateLimitPlugin from "./shared/plugins/rate-limit.plugin";
import redisPlugin from "./shared/plugins/redis.plugin";
import schedulePlugin from "./shared/plugins/schedule.plugin";
import swaggerPlugin from "./shared/plugins/swagger.plugin";

function getLoggerOptions() {
	if (process.stdout.isTTY) {
		return {
			level: env.LOG_LEVEL,
			transport:
				env.NODE_ENV === "development"
					? {
							target: "pino-pretty",
							options: {
								colorize: true,
								ignore: "pid,hostname",
								translateTime: "SYS:HH:MM:ss",
							},
						}
					: undefined,
		};
	}
	return { level: env.LOG_LEVEL ?? "silent" };
}

export async function AppServer() {
	const app = Fastify({
		logger: getLoggerOptions(),
	}).withTypeProvider<ZodTypeProvider>();

	// 1. FUNDAÇÃO (DI, Logger e Suporte a Middlewares)
	await app.register(awilixPlugin);
	await app.register(loggerPlugin);
	await app.register(middlePlugin);

	// 2. INFRAESTRUTURA (Redis primeiro para o Rate Limit usar)
	await app.register(redisPlugin);

	// 2. SEGURANÇA GLOBAL (Devem rodar antes de qualquer processamento)
	// O CORS precisa ser um dos primeiros para responder OPTIONS requests
	await app.register(corsPlugin);
	await app.register(helmetPlugin);
	await app.register(rateLimitPlugin);

	// 3. DOCUMENTAÇÃO (Deve vir antes dos módulos, mas após segurança básica)
	await app.register(swaggerPlugin);

	// 4. PARSERS E INFRA (Preparação do Request)
	await app.register(multipartPlugin); // Para lidar com arquivos
	await app.register(cookiePlugin); // Para parsear cookies antes das rotas
	await app.register(schedulePlugin);

	// 5. TRATAMENTO DE ERROS
	app.setErrorHandler(errorHandler);

	// 6. MÓDULOS DE NEGÓCIO
	const appModule = new AppModule();
	await appModule.register(app);
	await appModule.bootstrap(app);

	return app;
}
\n\n===== FILE: ./src/server.ts =====
import "dotenv/config";
import closeWithGrace from "close-with-grace";
import { AppServer } from "./app";
import { healthRoute } from "./infrastructure/health/health.route";
import { env } from "./shared/config";

const app = await AppServer();

await app.register(healthRoute);

closeWithGrace({ delay: 500 }, async ({ err }) => {
	if (err) app.log.error(err);
	await app.diContainer.dispose();
	await app.close();
});

try {
	await app.listen({ port: env.PORT, host: env.HOST });
	app.log.info(`🚀 Server running on http://${env.HOST}:${env.PORT}`);
	app.log.info(`📚 Docs: http://${env.HOST}:${env.PORT}/docs`);
} catch (err) {
	app.log.error(err);
	process.exit(1);
}
\n\n===== FILE: ./src/shared/plugins/awilix.plugin.ts =====
import { fastifyAwilixPlugin } from "@fastify/awilix";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function awilixPlugin(app: FastifyInstance) {
	await app.register(fastifyAwilixPlugin, {
		disposeOnClose: true,
		disposeOnResponse: true,
		strictBooleanEnforced: true,
		asyncInit: true,
		asyncDispose: true,
		eagerInject: true,
	});
}

export default fp(awilixPlugin, {
	name: "awilix-plugin",
});
\n\n===== FILE: ./src/shared/plugins/logger.plugin.ts =====
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function loggerPlugin(app: FastifyInstance) {
	app.addHook("onRequest", async (request) => {
		request.log.info(
			{
				method: request.method,
				url: request.url,
				ip: request.ip,
				userAgent: request.headers["user-agent"],
			},
			"Incomming request",
		);
	});

	app.addHook("onResponse", async (request, reply) => {
		const responseTime = reply.elapsedTime.toFixed(2);

		request.log.info(
			{
				method: request.method,
				url: request.url,
				statusCode: reply.statusCode,
				responseTime: `${responseTime}ms`,
			},
			"Request completed",
		);
	});
}

export default fp(loggerPlugin, {
	name: "logger-plugin",
});
\n\n===== FILE: ./src/shared/plugins/schedule.plugin.ts =====
import fastifySchedule from "@fastify/schedule";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function schedulePlugin(app: FastifyInstance) {
	await app.register(fastifySchedule, {});
}

export default fp(schedulePlugin, {
	name: "schedule-plugin",
});
\n\n===== FILE: ./src/shared/plugins/redis.plugin.ts =====
import { fastifyRedis } from "@fastify/redis";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import { env } from "../config";

async function redisPlugin(app: FastifyInstance) {
	await app.register(fastifyRedis, {
		url: env.REDIS_URL,
	});
}

export default fp(redisPlugin, {
	name: "redis-plugin",
});
\n\n===== FILE: ./src/shared/plugins/swagger.plugin.ts =====
import fastifySwagger from "@fastify/swagger";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import {
	createJsonSchemaTransform,
	serializerCompiler,
	validatorCompiler,
} from "fastify-type-provider-zod";
import { env } from "../config/index.js";


async function swaggerPlugin(app: FastifyInstance) {
	app.setValidatorCompiler(validatorCompiler);
	app.setSerializerCompiler(serializerCompiler);

	await app.register(fastifySwagger, {
		openapi: {
			info: {
				title: "API MasterSindico",
				description: "Docs API MasterSindico",
				version: "1.0.0",
			},
			servers: [
				{
					url: `http://${env.HOST}:${env.PORT}`,
					description: "Development server",
				},
			],
		},
		transform: createJsonSchemaTransform({ skipList: [] }),
	});

	await app.register(import("@scalar/fastify-api-reference"), {
		routePrefix: "/docs",
		configuration: {
			theme: "purple",
		},
	});
}

export default fp(swaggerPlugin, {
	name: "swagger-plugin",
});
\n\n===== FILE: ./src/shared/plugins/cookie.plugin.ts =====
import fastifyCookie from "@fastify/cookie";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import { env } from "../config/index.js";

async function cookiePlugin(app: FastifyInstance) {
	await app.register(fastifyCookie, {
		secret: env.COOKIE_SECRET,
		algorithm: "SHA256",
	});
}

export default fp(cookiePlugin, {
	name: "cookie-plugin",
});
\n\n===== FILE: ./src/shared/plugins/multipart.plugin.ts =====
import fastifyMultipart from "@fastify/multipart";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function multipartPlugin(app: FastifyInstance) {
	await app.register(fastifyMultipart, {
		limits: {
			fileSize: 100 * 1024 * 1024, // 100 MB
		},
	});
}

export default fp(multipartPlugin, {
	name: "multipart-plugin",
});
\n\n===== FILE: ./src/shared/plugins/rate-limit.plugin.ts =====
import fastifyRateLimit from "@fastify/rate-limit";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function rateLimitPlugin(app: FastifyInstance) {
	await app.register(fastifyRateLimit, {
		global: true,
		max: 100,
		timeWindow: "1 minute",
		redis: app.redis,
		cache: 5000,
		continueExceeding: false,
		skipOnError: true,
		keyGenerator: (request) => {
			// Identifica por IP ou pelo ID do usuário se estiver logado
			return request.user?.getId() || request.ip;
		},
		errorResponseBuilder: (_request, context) => {
			return {
				success: false,
				code: "TOO_MANY_REQUESTS",
				message: `Calma lá, irmão! Você atingiu o limite. Tente novamente em ${context.after}.`,
			};
		},
	});
}

export default fp(rateLimitPlugin, {
	name: "rate-limit-plugin",
	dependencies: ["redis-plugin"],
});
\n\n===== FILE: ./src/shared/plugins/cors.plugin.ts =====
import fastifyCors from "@fastify/cors";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import { env } from "../config/index.js";

async function corsPlugin(app: FastifyInstance) {
	await app.register(fastifyCors, {
		origin: env.CORS_ORIGIN === "*" ? true : env.CORS_ORIGIN.split(","),
		methods: ["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],
		allowedHeaders: ["Content-Type", "Authorization"],
		credentials: true,
	});
}

export default fp(corsPlugin, {
	name: "cors-plugin",
});
\n\n===== FILE: ./src/shared/plugins/helmet.plugin.ts =====
import fastifyHelmet from "@fastify/helmet";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";
import { env } from "../config/index.js";

async function helmetPlugin(app: FastifyInstance) {
	await app.register(fastifyHelmet, {
		global: true,
		contentSecurityPolicy: {
			directives: {
				defaultSrc: ["'self'"],
				styleSrc: ["'self'", "'unsafe-inline'"], // Swagger precisa
				scriptSrc: ["'self'", "'unsafe-inline'"], // Swagger precisa
				imgSrc: ["'self'", "data:", "https:"],
				connectSrc: ["'self'"],
				fontSrc: ["'self'", "https:", "data:"],
				objectSrc: ["'none'"],
				mediaSrc: ["'self'"],
				frameSrc: ["'none'"],
			},
		},
		...(env.NODE_ENV === "development" && {
			contentSecurityPolicy: false,
		}),
	});
}

export default fp(helmetPlugin, {
	name: "helmet-plugin",
});
\n\n===== FILE: ./src/shared/plugins/middle.plugin.ts =====
import fastifyMiddie from "@fastify/middie";
import type { FastifyInstance } from "fastify";
import fp from "fastify-plugin";

async function fastifyMiddiePlugin(app: FastifyInstance) {
	await app.register(fastifyMiddie, {});
}

export default fp(fastifyMiddiePlugin, {
	name: "middle-plugin",
});
\n\n===== FILE: ./src/shared/config/index.ts =====
import dotenv from "dotenv";
import { z } from "zod";

dotenv.config();
const envSchema = z.object({
	NODE_ENV: z
		.enum(["development", "test", "production"])
		.default("development"),
	PORT: z.coerce.number().default(8000),
	HOST: z.string().default("0.0.0.0"),
	LOG_LEVEL: z
		.enum(["fatal", "error", "warn", "info", "debug", "trace", "silent"])
		.default("info"),

	// Database
	DATABASE_URL: z.string(),

	// Redis
	REDIS_URL: z.string(),

	// CORS
	CORS_ORIGIN: z.string().default("*"),

	// Cookies
	COOKIE_SECRET: z.string().min(32),

	// Auth & Session
	JWT_SECRET: z.string().min(32),

	// Tempo total de vida da sessão (ex: 7 dias)
	SESSION_EXPIRATION_DAYS: z.coerce.number().int().positive().default(7),

	// Quando faltar X dias, a gente renova a sessão (ex: 2 dias)
	SESSION_REFRESH_THRESHOLD_DAYS: z.coerce.number().int().positive().default(2),

	SESSION_MAX_ACTIVE_SESSIONS: z.coerce.number().int().positive().default(5),
	VERIFICATION_EXPIRATION_MINS: z.coerce.number().int().positive().default(1),

	// OAuth
	// Google Provider OAuth
	GOOGLE_CLIENT_ID: z.string(),
	GOOGLE_CLIENT_SECRET: z.string(),
	GOOGLE_CALLBACK_URL: z.url(),

	// Email Providers
	// PRIMARY: RESEND
	RESEND_SENDER_EMAIL: z.email(),
	RESEND_API_KEY: z.string(),

	// SECONDARY: GOOGLE/GMAIL (SMTP de backup)
	GMAIL_USER: z.email(),
	GMAIL_APP_PASSWORD: z.string(), // Não é a senha de login!
	GMAIL_SENDER_EMAIL: z.email(),

	FRONTEND_URL: z.url(),

	// Stripe
	STRIPE_KEY_PUB: z.string(),
	STRIPE_KEY_PRIV: z.string(),
});

export type Env = z.infer<typeof envSchema>;
function validateEnv(): Env {
	const result = envSchema.safeParse(process.env);

	if (!result.success) {
		console.error("❌ Invalid environment variables:");
		console.error(
			"Invalid environment variables:",
			z.treeifyError(result.error),
		);
		process.exit(1);
	}

	return result.data;
}
export const env = validateEnv();
\n\n===== FILE: ./src/shared/module/module.interface.ts =====
import type { FastifyInstance } from "fastify";

export interface Module {
	register(app: FastifyInstance): Promise<void>;
	bootstrap?(app: FastifyInstance): Promise<void>;
}
\n\n===== FILE: ./src/shared/helpers/response.helper.ts =====
import type {
	ApiError,
	ApiSuccess,
	PaginatedResponse,
} from "../types/api-response";

/**
 * Helper para resposta de sucesso
 */
export const success = <T>(data: T): ApiSuccess<T> => ({
	success: true,
	data,
});

/**
 * Helper para resposta de erro
 */
export const error = (message: string, code?: string): ApiError => ({
	success: false,
	error: message,
	code,
});

/**
 * Helper para resposta paginada
 */
export const paginated = <T>(
	data: T[],
	page: number,
	limit: number,
	total?: number,
): PaginatedResponse<T> => ({
	success: true,
	data,
	pagination: { page, limit, total },
});

/**
 * Códigos de erro padrão
 */
export const ErrorCodes = {
	// Genéricos
	NOT_FOUND: "NOT_FOUND",
	ALREADY_EXISTS: "ALREADY_EXISTS",
	VALIDATION_ERROR: "VALIDATION_ERROR",
	INTERNAL_ERROR: "INTERNAL_ERROR",

	// Auth
	UNAUTHORIZED: "UNAUTHORIZED",
	FORBIDDEN: "FORBIDDEN",
	INVALID_CREDENTIALS: "INVALID_CREDENTIALS",
	TOKEN_EXPIRED: "TOKEN_EXPIRED",
} as const;

/**
 * Mensagens de erro padrão
 */
export const ErrorMessages = {
	[ErrorCodes.NOT_FOUND]: "Recurso não encontrado",
	[ErrorCodes.ALREADY_EXISTS]: "Recurso já existe",
	[ErrorCodes.VALIDATION_ERROR]: "Dados inválidos",
	[ErrorCodes.INTERNAL_ERROR]: "Erro interno do servidor",

	[ErrorCodes.UNAUTHORIZED]: "Não autorizado",
	[ErrorCodes.FORBIDDEN]: "Acesso negado",
	[ErrorCodes.INVALID_CREDENTIALS]: "Credenciais inválidas",
	[ErrorCodes.TOKEN_EXPIRED]: "Token expirado",
} as const;
\n\n===== FILE: ./src/shared/types/api-response.ts =====
import { z } from "zod";

// Schemas Zod para responses
export const apiSuccessSchema = <T extends z.ZodTypeAny>(dataSchema: T) =>
	z.object({
		success: z.literal(true),
		data: dataSchema,
	});

export const apiErrorSchema = z.object({
	success: z.literal(false),
	error: z.string(),
	code: z.string().optional(),
});

export const paginatedResponseSchema = <T extends z.ZodTypeAny>(
	itemSchema: T,
) =>
	z.object({
		success: z.literal(true),
		data: z.array(itemSchema),
		pagination: z.object({
			page: z.number(),
			limit: z.number(),
			total: z.number().optional(),
		}),
	});

// Tipos TypeScript (inferidos do Zod)
export type ApiSuccess<T> = {
	success: true;
	data: T;
};

export type ApiError = {
	success: false;
	error: string;
	code?: string;
};

export type PaginatedResponse<T> = {
	success: true;
	data: T[];
	pagination: {
		page: number;
		limit: number;
		total?: number;
	};
};

export type ApiResponse<T> = ApiSuccess<T> | ApiError;
\n\n===== FILE: ./src/shared/types/fastify.d.ts =====
import type { Session } from "../../modules/auth/domain/entities/session.entity.ts";
import type { User } from "../../modules/user/domain/entities/user.entity.ts";

declare module "fastify" {
	interface FastifyRequest {
		user: User;
		session: Session;
	}
}
\n\n===== FILE: ./src/shared/types/awilix.d.ts =====
import "@fastify/awilix";

declare module "@fastify/awilix" {
	interface Cradle {
		// Core
		logger;
		database;
		unitOfWork;

		// Repositories - Auth
		userRepository;
		accountRepository;
		sessionRepository;
		verificationRepository;

		// Repositories - Profile
		residentProfileRepository;
		syndicProfileRepository;
		enterpriseProfileRepository;
		localCompanyProfileRepository;
		marketingProfileRepository;

		// Providers
		emailProvider;
		resend;
		gmail;

		// Tasks
		verificationCleanupTask;
		sessionCleanupTask;

		// Services
		usernameGenerator;
		authService;
		jwtService;

		// Use Cases - Auth
		signUpUseCase;
		signInUseCase;
		signOutUseCase;
		sendVerificationCodeUseCase;
		verifyCodeUseCase;
		forgotPasswordUseCase;
		resetPasswordUseCase;
		getSessionUseCase;
		revokeSessionUseCase;
		revokeOtherSessionsUseCase;
		revokeAllSessionsUseCase;
		listSessionsUseCase;

		// Use Cases - Profile
		// Create
		createResidentProfileUseCase;
		createSyndicProfileUseCase;
		createEnterpriseProfileUseCase;
		createLocalCompanyProfileUseCase;
		createMarketingProfileUseCase;

		// Update
		updateResidentProfileUseCase;
		updateSyndicProfileUseCase;
		updateEnterpriseProfileUseCase;
		updateLocalCompanyProfileUseCase;
		updateMarketingProfileUseCase;

		// Get
		getProfileByUserIdUseCase;
	}
}
\n\n===== FILE: ./src/shared/utils/id-generator.ts =====
import { customAlphabet } from "nanoid";
import { v7 as uuidv7 } from "uuid";

/**
 * Gera um ID único usando UUID v7
 *
 * UUID v7 é sortable (pode ordenar cronologicamente) e contém timestamp,
 * sendo ideal para IDs de banco de dados.
 *
 * Formato: xxxxxxxx-xxxx-7xxx-xxxx-xxxxxxxxxxxx
 * Exemplo: 018d3f1c-073e-7890-abcd-1234567890ab
 *
 * Benefícios:
 * - ✅ Sortable (ordenação cronológica)
 * - ✅ Timestamp embutido (primeiros 48 bits)
 * - ✅ Compatível com Jest/CommonJS
 * - ✅ Padrão RFC 9562
 * - ✅ Seguro criptograficamente
 */
export function generateId(): string {
	return uuidv7();
}

/**
 * Remove hífens do UUID para formato compacto
 *
 * @param id - UUID com hífens
 * @returns UUID sem hífens (32 caracteres)
 *
 * Exemplo: 018d3f1c073e7890abcd1234567890ab
 */
export function compactId(id?: string): string {
	return (id || generateId()).replace(/-/g, "");
}

/**
 * Gera ID com prefixo customizado (estilo CUID2)
 *
 * @param prefix - Prefixo opcional (ex: 'usr', 'ses', 'acc')
 * @returns ID prefixado
 *
 * Exemplo: usr_018d3f1c073e7890abcd1234567890ab
 */
export function generatePrefixedId(prefix: string): string {
	return `${prefix}_${compactId()}`;
}

const ALPHABET = "23456789abcdefghjkmnpqrstuvwxyz";
const SLUG_LENGTH = 10;

const genSlug = customAlphabet(ALPHABET, SLUG_LENGTH);

export function generateSlug(): string {
	return genSlug();
}

export function isValidSlug(slug: string): boolean {
	if (slug.length !== SLUG_LENGTH) return false;
	return slug.split("").every((char) => ALPHABET.includes(char));
}

export function generatePrefixedSlug(prefix: "p" | "c" = "p"): string {
	return `${prefix}-${generateSlug()}`;
}
\n\n===== FILE: ./src/shared/utils/request-parser.ts =====
/** biome-ignore-all lint/complexity/noStaticOnlyClass: <> */
import type { FastifyRequest } from "fastify";

export class RequestParser {
	static extractToken(request: FastifyRequest): string | null {
		return (
			request.cookies?.session_token ||
			request.headers.authorization?.replace("Bearer ", "") ||
			null
		);
	}

	static getMetadata(request: FastifyRequest) {
		const userAgent = request.headers["user-agent"]?.toLowerCase() || "";
		return {
			ip: request.ip,
			userAgent,
			isMobile: /mobile|android|iphone|ipad|dart|okhttp/.test(userAgent),
		};
	}
}
\n\n===== FILE: ./src/shared/contracts/auth-metadata.ts =====
export interface AuthMetadata {
	ipAddress: string;
	userAgent?: string;
}
\n\n===== FILE: ./src/shared/providers/email.provider.ts =====
export interface IEmailProvider {
	/**
	 * Envia o código de 6 dígitos para validação inicial ou MFA
	 */
	sendVerificationCode(
		to: string,
		code: string,
		expiresAt: Date,
	): Promise<void>;

	/**
	 * Notifica o usuário que a conta foi criada (Boas-vindas)
	 */
	sendAccountCreatedNotification(to: string, username: string): Promise<void>;

	/**
	 * Envia o código de 6 dígitos para troca de senha ou MFA
	 */
	sendPasswordResetCode(
		to: string,
		code: string,
		expiresAt: Date,
	): Promise<void>;

	/**
	 * Alerta de segurança sobre novo acesso
	 */
	sendNewLoginNotification(
		to: string,
		username: string,
		loginDate: Date,
		ipAddress?: string,
		userAgent?: string,
	): Promise<void>;

	/**
	 * Retorna o nome do provedor atual (útil para logs e debug)
	 */
	getName(): string;
}
\n\n===== FILE: ./src/shared/hooks/authenticate.hook.ts =====
import type { FastifyReply, FastifyRequest } from "fastify";
import { UnauthorizedError } from "../../core/errors";

export async function authenticate(
	request: FastifyRequest,
	_reply: FastifyReply,
) {
	const { getSessionUseCase, jwtService, userRepository, sessionRepository } =
		request.diScope.cradle;

	const token = jwtService.extractToken(request);

	if (!token) {
		throw new UnauthorizedError("Token de autenticação ausente");
	}

	try {
		// Obter DTOs
		const { session: sessionDTO, user: userDTO } =
			await getSessionUseCase.execute({ token });

		// ✅ Buscar as entidades completas do repositório
		const user = await userRepository.findById(userDTO.id);
		const session = await sessionRepository.findOneById(sessionDTO.id);

		if (!user || !session) {
			throw new UnauthorizedError("Sessão ou usuário inválido");
		}

		// ✅ Agora injeta as ENTIDADES, não os DTOs
		request.user = user;
		request.session = session;

		if (sessionDTO.token !== token) {
			_reply.header("x-refresh-token", sessionDTO.token);
		}
	} catch (_error) {
		throw new UnauthorizedError("Sessão inválida ou expirada");
	}
}
\n\n===== FILE: ./src/infrastructure/database/drizzle/drizzle.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asClass, asValue, Lifetime } from "awilix";
import type { FastifyInstance } from "fastify";
import type { Module } from "../../../shared/module/module.interface";
import { db } from "./client";
import { DrizzleUnitOfWork } from "./unit-of-work/unit-of-work";

export class DrizzleModule implements Module {
	async register(app: FastifyInstance): Promise<void> {
		diContainer.register({
			// 1. O Banco é SINGLETON (vive pra sempre, uma conexão só)
			database: asValue(db),

			// 2. O UoW é SCOPED (o Awilix cria um novo automaticamente a cada request)
			// Não precisa de hook 'onRequest', o plugin fastify-awilix já faz essa mágica.
			unitOfWork: asClass(DrizzleUnitOfWork, {
				lifetime: Lifetime.SCOPED,
			}),
		});
		app.log.info("✅ Drizzle database and UnitOfWork registered");
	}
	async bootstrap(app: FastifyInstance): Promise<void> {
		try {
			await db.execute("SELECT 1");
			app.log.info("✅ Database connection established successfully.");
		} catch (error: unknown) {
			const errorMessage =
				error instanceof Error ? error.message : "Unknown error";
			app.log.error(`❌ Failed to connect to the database: ${errorMessage}`);
			throw new Error(
				`Database connection failed during bootstrap: ${errorMessage}`,
			);
		}
	}
}
\n\n===== FILE: ./src/infrastructure/database/drizzle/client.ts =====
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import { env } from "../../../shared/config";
import * as schema from "./schema";
import { relations } from "./schema/relations";

const queryClient = postgres(env.DATABASE_URL, {
	max: 20,
	idle_timeout: 30000,
	connect_timeout: 2000,
});

export const db = drizzle({
	client: queryClient,
	schema: {
		...schema,
	},
	relations: {
		...relations,
	},
	casing: "snake_case",
	logger: env.NODE_ENV === "development", // 👈 Log só em dev
});

export type DrizzleClient = typeof db;
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/auth/index.ts =====
export * from "./accounts";
export * from "./sessions";
export * from "./verifications";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/auth/verifications.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";

export const verifications = pgTable(
	"verifications",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),
		identifier: text("identifier").notNull(),
		value: text("value").notNull(),
		expiresAt: timestamp("expires_at").notNull(),
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(table) => [index("verifications_identifier_idx").on(table.identifier)],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/auth/accounts.ts =====
import * as p from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { users } from "../users/users";

export const accounts = p.pgTable(
	"accounts",
	{
		id: p
			.uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),
		accountId: p.text("account_id").notNull(),
		providerId: p.text("provider_id").notNull(),
		userId: p
			.uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),

		accessToken: p.text("access_token"),
		refreshToken: p.text("refresh_token"),
		idToken: p.text("id_token"),

		accessTokenExpiresAt: p.timestamp("access_token_expires_at"),
		refreshTokenExpiresAt: p.timestamp("refresh_token_expires_at"),
		scope: p.text("scope"),

		password: p.text("password"), // Hash para local

		createdAt: p.timestamp("created_at").notNull(),
		updatedAt: p
			.timestamp("updated_at")
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: p.timestamp("deleted_at"),
	},
	(t) => [
		p.index("accounts_user_id_idx").on(t.userId),
		p.index("accounts_provider_idx").on(t.providerId, t.accountId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/auth/sessions.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { users } from "../users";

export const sessions = pgTable(
	"sessions",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		expiresAt: timestamp("expires_at").notNull(),
		ipAddress: text("ip_address"),
		userAgent: text("user_agent"),
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		index("sessions_expires_at_idx").on(t.expiresAt),
		index("sessions_user_id_idx").on(t.userId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/users/index.ts =====
export * from "./users";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/users/users.ts =====
import { sql } from "drizzle-orm";
import * as p from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { userRoles } from "../enums";

export const users = p.pgTable(
	"users",
	{
		id: p
			.uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		name: p.text("name").notNull(),
		email: p.text("email").notNull(),
		emailVerified: p.boolean("email_verified").default(false).notNull(),
		username: p.text("username").notNull(),
		image: p.text("image"),
		phone: p.text("phone"),

		role: userRoles("role").default("none").notNull(),

		banned: p.boolean("banned").default(false),
		banReason: p.text("ban_reason"),
		banExpires: p.timestamp("ban_expires"),

		createdAt: p
			.timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: p
			.timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: p.timestamp("deleted_at"),
	},
	(t) => [
		p
			.uniqueIndex("users_username_idx")
			.on(t.username)
			.where(sql`${t.deletedAt} IS NULL`),
		p
			.uniqueIndex("users_email_idx")
			.on(t.email)
			.where(sql`${t.deletedAt} IS NULL`),
		p.index("users_role_idx").on(t.role),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/index.ts =====
export * from "./enterprise";
export * from "./localCompany";
export * from "./marketing";
export * from "./resident";
export * from "./syndic";


\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/resident.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { profileStatus } from "../enums";
import { users } from "../users/users";

export const residentProfile = pgTable(
	"resident_profile",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.unique()
			.references(() => users.id, { onDelete: "cascade" }),

		birthDate: timestamp("birth_date").notNull(),
		cpf: text("cpf").notNull().unique(),

		// Endereço
		street: text("street").notNull(),
		number: text("number").notNull(),
		block: text("block"),
		district: text("district").notNull(),
		city: text("city").notNull(),
		state: text("state").notNull(),
		zipCode: text("zip_code").notNull(),

		// Vínculo com condomínio
		buildingNameOrCode: text("building_name_or_code"),

		status: profileStatus("status").default("incomplete").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("resident_profile_user_id_idx").on(t.userId),
		index("resident_profile_cpf_idx").on(t.cpf),
		index("resident_profile_status_idx").on(t.status),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/enterprise.ts =====
import {
	boolean,
	index,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { profileStatus } from "../enums";
import { users } from "../users/users";

export const enterpriseProfile = pgTable(
	"enterprise_profile",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.unique()
			.references(() => users.id, { onDelete: "cascade" }),

		// Dados da Empresa
		companyName: text("company_name").notNull(), // Razão social
		companyTradeName: text("company_trade_name").notNull(), // Nome fantasia
		cnpj: text("cnpj").notNull().unique(),
		foundationDate: timestamp("foundation_date").notNull(),

		// Representante Legal
		legalRepresentativeName: text("legal_representative_name").notNull(),
		legalRepresentativeEmail: text("legal_representative_email").notNull(),
		legalRepresentativePhone: text("legal_representative_phone"),

		// Contato Comercial
		commercialEmail: text("commercial_email").notNull(),
		commercialPhone: text("commercial_phone").notNull(),

		// Endereço Comercial
		street: text("street").notNull(),
		number: text("number").notNull(),
		block: text("block"),
		district: text("district").notNull(),
		city: text("city").notNull(),
		state: text("state").notNull(),
		zipCode: text("zip_code").notNull(),

		// Financeiro
		financeContactName: text("finance_contact_name").notNull(),
		financeContactPhone: text("finance_contact_phone").notNull(),
		financeContactEmail: text("finance_contact_email").notNull(),

		// Atuação
		operatingCities: text("operating_cities").array(),

		// Informações Complementares
		hasCivilLiabilityInsurance: boolean("has_civil_liability_insurance").default(false),
		hasLegalAdvice: boolean("has_legal_advice").default(false),
		hasAccountingAdvice: boolean("has_accounting_advice").default(false),
		hasWorkSafetyAdvice: boolean("has_work_safety_advice").default(false),
		hasTechnicalManager: boolean("has_technical_manager").default(false),
		hasProfessionalCouncilRegularity: boolean("has_professional_council_regularity").default(false),
		hasLgpdCompliance: boolean("has_lgpd_compliance").default(false),
		hasNr1Compliance: boolean("has_nr1_compliance").default(false),
		hasComplianceProgram: boolean("has_compliance_program").default(false),

		// ISOs
		hasIso9001: boolean("has_iso_9001").default(false),
		hasIso14001: boolean("has_iso_14001").default(false),
		hasIso45001: boolean("has_iso_45001").default(false),
		hasIso37001: boolean("has_iso_37001").default(false),
		hasIso19600_37301: boolean("has_iso_19600_37301").default(false),

		// ESG e Selos
		hasEsg: boolean("has_esg").default(false),
		hasGreenSeal: boolean("has_green_seal").default(false),
		hasChildFriendlySeal: boolean("has_child_friendly_seal").default(false),
		hasCarbonFreeSeal: boolean("has_carbon_free_seal").default(false),
		hasEuRecicloSeal: boolean("has_eu_reciclo_seal").default(false),
		hasReclameAquiSeal: boolean("has_reclame_aqui_seal").default(false),
		hasCipa: boolean("has_cipa").default(false),

		// Campos abertos
		nrs: text("nrs"),
		otherCertifications: text("other_certifications"),
		ownCertifications: text("own_certifications"),
		awards: text("awards"),

		// Conteúdo Complementar
		institutionalDescription: text("institutional_description"),
		portfolioUrl: text("portfolio_url"),

		status: profileStatus("status").default("incomplete").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("enterprise_profile_user_id_idx").on(t.userId),
		index("enterprise_profile_cnpj_idx").on(t.cnpj),
		index("enterprise_profile_status_idx").on(t.status),
	],
);

\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/marketing.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { profileStatus } from "../enums";
import { users } from "../users/users";

export const marketingProfile = pgTable(
	"marketing_profile",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.unique()
			.references(() => users.id, { onDelete: "cascade" }),

		// Dados da Empresa
		companyName: text("company_name").notNull(), // Razão social
		companyTradeName: text("company_trade_name").notNull(), // Nome fantasia
		cnpj: text("cnpj").notNull().unique(),
		foundationDate: timestamp("foundation_date").notNull(),

		// Representante Legal
		legalRepresentativeName: text("legal_representative_name").notNull(),
		legalRepresentativeEmail: text("legal_representative_email").notNull(),
		legalRepresentativePhone: text("legal_representative_phone"),

		// Contato Comercial
		commercialEmail: text("commercial_email").notNull(),
		commercialPhone: text("commercial_phone").notNull(),

		// Endereço Comercial
		street: text("street").notNull(),
		number: text("number").notNull(),
		block: text("block"),
		district: text("district").notNull(),
		city: text("city").notNull(),
		state: text("state").notNull(),
		zipCode: text("zip_code").notNull(),

		// Financeiro
		financeContactName: text("finance_contact_name").notNull(),
		financeContactPhone: text("finance_contact_phone").notNull(),
		financeContactEmail: text("finance_contact_email").notNull(),

		// Atuação
		operatingCities: text("operating_cities").array(),

		status: profileStatus("status").default("incomplete").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("marketing_profile_user_id_idx").on(t.userId),
		index("marketing_profile_cnpj_idx").on(t.cnpj),
		index("marketing_profile_status_idx").on(t.status),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/syndic.ts =====
import {
	boolean,
	index,
	integer,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { profileStatus, syndicType, taxIdType } from "../enums";
import { users } from "../users/users";

export const syndicProfile = pgTable(
	"syndic_profile",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.unique()
			.references(() => users.id, { onDelete: "cascade" }),

		birthDate: timestamp("birth_date").notNull(),
		document: text("document").notNull().unique(), // CPF ou CNPJ
		documentType: taxIdType("document_type").notNull(), // br_cpf ou br_cnpj (Stripe-compatible)

		// Endereço Profissional
		street: text("street").notNull(),
		number: text("number").notNull(),
		block: text("block"),
		district: text("district").notNull(),
		city: text("city").notNull(),
		state: text("state").notNull(),
		zipCode: text("zip_code").notNull(),

		// Atuação
		type: syndicType("type").notNull(),
		experienceYears: integer("experience_years").notNull(),
		buildingsCount: integer("buildings_count").notNull(),
		operatingCities: text("operating_cities").array(),

		// Informações Complementares
		hasAbracsMember: boolean("has_abracs_member").default(false),
		hasCivilLiabilityInsurance: boolean("has_civil_liability_insurance").default(false),
		hasLegalAdvice: boolean("has_legal_advice").default(false),
		hasAccountingAdvice: boolean("has_accounting_advice").default(false),
		hasWorkSafetyAdvice: boolean("has_work_safety_advice").default(false),
		hasLgpdCompliance: boolean("has_lgpd_compliance").default(false),
		hasNr1Compliance: boolean("has_nr1_compliance").default(false),
		hasComplianceProgram: boolean("has_compliance_program").default(false),
		hasReclameAquiSeal: boolean("has_reclame_aqui_seal").default(false),

		// Campos abertos
		otherCertifications: text("other_certifications"),
		awards: text("awards"),

		// Dados Níveis 2 e 3
		miniBio: text("mini_bio"),
		educationAndCertifications: text("education_and_certifications"),
		linkedAdministrators: text("linked_administrators"),

		status: profileStatus("status").default("incomplete").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("syndic_profile_user_id_idx").on(t.userId),
		index("syndic_profile_document_idx").on(t.document),
		index("syndic_profile_type_idx").on(t.type),
		index("syndic_profile_status_idx").on(t.status),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/profiles/localCompany.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { profileStatus, taxIdType } from "../enums";
import { users } from "../users/users";

export const localCompanyProfile = pgTable(
	"local_company_profile",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.unique()
			.references(() => users.id, { onDelete: "cascade" }),

		// Dados da Empresa
		companyName: text("company_name").notNull(), // Razão social
		companyTradeName: text("company_trade_name").notNull(), // Nome fantasia
		document: text("document").notNull().unique(), // CPF ou CNPJ
		documentType: taxIdType("document_type").notNull(), // br_cpf ou br_cnpj (Stripe-compatible)
		foundationDate: timestamp("foundation_date").notNull(),

		// Representante Legal
		legalRepresentativeName: text("legal_representative_name").notNull(),
		legalRepresentativeEmail: text("legal_representative_email").notNull(),
		legalRepresentativePhone: text("legal_representative_phone"),

		// Contato Comercial
		commercialEmail: text("commercial_email").notNull(),
		commercialPhone: text("commercial_phone").notNull(),

		// Endereço Comercial
		street: text("street").notNull(),
		number: text("number").notNull(),
		block: text("block"),
		district: text("district").notNull(),
		city: text("city").notNull(),
		state: text("state").notNull(),
		zipCode: text("zip_code").notNull(),

		// Financeiro
		financeContactName: text("finance_contact_name").notNull(),
		financeContactPhone: text("finance_contact_phone").notNull(),
		financeContactEmail: text("finance_contact_email").notNull(),

		// Até 3 fotos
		photos: text("photos").array(),

		status: profileStatus("status").default("incomplete").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("local_company_profile_user_id_idx").on(t.userId),
		index("local_company_profile_document_idx").on(t.document),
		index("local_company_profile_status_idx").on(t.status),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/videos/videos.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { videoStatus, videoType } from "../enums";
import { users } from "../users/users";

interface VideoProviderMetadata {
	muxAssetId?: string;
	muxPlaybackId?: string;
	cloudflareUid?: string;
	cloudflarePlaybackUrl?: string;
	bunnyVideoId?: string;
	bunnyLibraryId?: string;
	originalFileName?: string;
	encodingProgress?: number;
	qualities?: string[];
}

export const videos = pgTable(
	"videos",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		authorUserId: uuid("author_user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),

		title: text("title").notNull(),
		description: text("description"),
		type: videoType("type").notNull(),
		status: videoStatus("status").default("pending").notNull(),

		// ===================================
		// PROVIDER-AGNOSTIC
		// ===================================
		provider: text("provider").notNull(),
		providerVideoId: text("provider_video_id").unique(),

		// URLs finais
		streamUrl: text("stream_url"),
		thumbnailUrl: text("thumbnail_url"),

		// Metadados técnicos
		durationSeconds: integer("duration_seconds"),
		aspectRatio: text("aspect_ratio"),

		// Metadata do provedor
		providerMetadata: jsonb("provider_metadata").$type<VideoProviderMetadata>(),

		// ===================================
		// CONTROLE DE ACESSO
		// ===================================
		isPublic: boolean("is_public").default(true),
		requiresAuth: boolean("requires_auth").default(false),

		// ===================================
		// CATEGORIZAÇÃO
		// ===================================
		serviceCategoryId: uuid("service_category_id"),

		// ===================================
		// MÉTRICAS
		// ===================================
		viewCount: integer("view_count").default(0),
		completionCount: integer("completion_count").default(0),
		likeCount: integer("like_count").default(0),
		commentCount: integer("comment_count").default(0),

		// ===================================
		// TRAVA TRIMESTRAL
		// ===================================
		canBeReplacedAfter: timestamp("can_be_replaced_after"),
		replacesVideoId: uuid("replaces_video_id"),

		// ===================================
		// MODERAÇÃO
		// ===================================
		isFlagged: boolean("is_flagged").default(false),
		flaggedAt: timestamp("flagged_at"),
		moderatedAt: timestamp("moderated_at"),
		moderatedByUserId: uuid("moderated_by_user_id"),
		moderationNote: text("moderation_note"),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("videos_author_idx").on(t.authorUserId),
		index("videos_type_idx").on(t.type),
		index("videos_status_idx").on(t.status),
		index("videos_provider_video_id_idx").on(t.providerVideoId),
		index("videos_category_idx").on(t.serviceCategoryId),
		index("videos_created_at_idx").on(t.createdAt),
		index("videos_author_type_idx").on(t.authorUserId, t.type),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/videos/video-views.ts =====
import {
	boolean,
	index,
	integer,
	pgTable,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { sessions } from "../auth/sessions";
import { users } from "../users/users";
import { videos } from "./videos";

export const videoViews = pgTable(
	"video_views",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		videoId: uuid("video_id")
			.notNull()
			.references(() => videos.id, { onDelete: "cascade" }),
		userId: uuid("user_id").references(() => users.id, { onDelete: "cascade" }),

		// Referencia session existente (não duplica dados)
		sessionId: uuid("session_id").references(() => sessions.id, {
			onDelete: "set null",
		}),

		// Progresso
		watchedSeconds: integer("watched_seconds").default(0),
		percentageWatched: integer("percentage_watched").default(0),
		completed: boolean("completed").default(false),

		viewedAt: timestamp("viewed_at")
			.$defaultFn(() => new Date())
			.notNull(),
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("video_views_video_idx").on(t.videoId),
		index("video_views_user_idx").on(t.userId),
		index("video_views_session_idx").on(t.sessionId),
		index("video_views_viewed_at_idx").on(t.viewedAt),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/videos/video-likes.ts =====
import {
	index,
	pgTable,
	timestamp,
	uniqueIndex,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { users } from "../users/users";
import { videos } from "./videos";

export const videoLikes = pgTable(
	"video_likes",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		videoId: uuid("video_id")
			.notNull()
			.references(() => videos.id, { onDelete: "cascade" }),
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("video_likes_video_idx").on(t.videoId),
		uniqueIndex("video_likes_user_video_idx").on(t.userId, t.videoId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/videos/video-comments.ts =====
import {
	index,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { users } from "../users/users";
import { videos } from "./videos";

export const videoComments = pgTable(
	"video_comments",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		videoId: uuid("video_id")
			.notNull()
			.references(() => videos.id, { onDelete: "cascade" }),
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),

		content: text("content").notNull(),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
		deletedAt: timestamp("deleted_at"),
	},
	(t) => [
		index("video_comments_video_idx").on(t.videoId),
		index("video_comments_user_idx").on(t.userId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/videos/index.ts =====
export * from "./video-comments";
export * from "./video-likes";
export * from "./video-views";
export * from "./videos";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/categories/service-categories.ts =====
import {
	boolean,
	index,
	integer,
	pgTable,
	text,
	timestamp,
	uniqueIndex,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";

export const serviceCategories = pgTable(
	"service_categories",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		slug: text("slug").unique().notNull(),
		name: text("name").notNull(),
		description: text("description"),

		// Hierarquia (self-reference para subcategorias)
		parentId: uuid("parent_id"),

		order: integer("order").default(0),
		isActive: boolean("is_active").default(true),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		index("service_categories_slug_idx").on(t.slug),
		index("service_categories_parent_idx").on(t.parentId),
		index("service_categories_active_idx").on(t.isActive),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/categories/enterprise-categories.ts =====
import {
	index,
	pgTable,
	timestamp,
	uniqueIndex,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { enterpriseProfile } from "../profiles/enterprise";
import { serviceCategories } from "./service-categories";

// Empresa → Categorias (sem limite)
export const enterpriseProfileCategories = pgTable(
	"enterprise_profile_categories",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		profileId: uuid("profile_id")
			.notNull()
			.references(() => enterpriseProfile.id, { onDelete: "cascade" }),
		categoryId: uuid("category_id")
			.notNull()
			.references(() => serviceCategories.id, { onDelete: "cascade" }),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("enterprise_profile_categories_profile_idx").on(t.profileId),
		index("enterprise_profile_categories_category_idx").on(t.categoryId),
		uniqueIndex("enterprise_profile_categories_unique").on(t.profileId, t.categoryId),
	],
);

// Empresa → Subcategorias (máximo 5, validação no app)
export const enterpriseProfileSubcategories = pgTable(
	"enterprise_profile_subcategories",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		profileId: uuid("profile_id")
			.notNull()
			.references(() => enterpriseProfile.id, { onDelete: "cascade" }),
		subcategoryId: uuid("subcategory_id")
			.notNull()
			.references(() => serviceCategories.id, { onDelete: "cascade" }),

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("enterprise_profile_subcategories_profile_idx").on(t.profileId),
		index("enterprise_profile_subcategories_subcategory_idx").on(t.subcategoryId),
		uniqueIndex("enterprise_profile_subcategories_unique").on(t.profileId, t.subcategoryId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/categories/index.ts =====
export * from "./service-categories";
export * from "./enterprise-categories";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/plan-features.ts =====
import { index, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { plans } from "./plans";

export const planFeatures = pgTable(
	"plan_features",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		planId: uuid("plan_id")
			.notNull()
			.references(() => plans.id, { onDelete: "cascade" }),

		featureKey: text("feature_key").notNull(), // "connect_me_quota"
		featureValue: text("feature_value").notNull(), // "4"
		featureType: text("feature_type").notNull(), // "integer" | "boolean" | "string"

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		index("plan_features_plan_idx").on(t.planId),
		index("plan_features_key_idx").on(t.featureKey),
		index("plan_features_plan_key_idx").on(t.planId, t.featureKey),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/permissions.ts =====
import { index, jsonb, pgTable, text, timestamp, uuid } from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { plans } from "./plans";

export const permissions = pgTable(
	"permissions",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		key: text("key").unique().notNull(), // "create_video"
		name: text("name").notNull(), // "Criar Vídeo"
		description: text("description"),
		category: text("category"), // "content" | "assembly" | "connect_me"

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("permissions_key_idx").on(t.key),
		index("permissions_category_idx").on(t.category),
	],
);

export const planPermissions = pgTable(
	"plan_permissions",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		planId: uuid("plan_id")
			.notNull()
			.references(() => plans.id, { onDelete: "cascade" }),
		permissionId: uuid("permission_id")
			.notNull()
			.references(() => permissions.id, { onDelete: "cascade" }),

		metadata: jsonb("metadata"), // Extra config per permission

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
	},
	(t) => [
		index("plan_permissions_plan_id_idx").on(t.planId),
		index("plan_permissions_permission_id_idx").on(t.permissionId),
		index("plan_permissions_plan_permission_idx").on(t.planId, t.permissionId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/payment-methods.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { cardBrand, paymentMethodStatus, paymentMethodType } from "../enums";
import { users } from "../users/users";

/**
 * Payment Methods - Métodos de pagamento salvos dos usuários
 *
 * Suporta múltiplos tipos de pagamento:
 * - card: Cartão de crédito/débito (suporta recorrência)
 * - pix: PIX (apenas pagamento único - expira em 24h)
 * - boleto: Boleto bancário (apenas pagamento único - 1-3 dias úteis)
 * - bank_transfer: Transferência bancária
 * - google_pay/apple_pay: Carteiras digitais (suportam recorrência)
 *
 * IMPORTANTE:
 * - PIX, Boleto e Transferência NÃO suportam recorrência na Stripe
 * - Nunca armazenar número completo do cartão (apenas last4)
 * - Dados sensíveis ficam no Stripe (provider)
 */
export const paymentMethods = pgTable(
	"payment_methods",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),

		// ===================================
		// TIPO E CONFIGURAÇÕES
		// ===================================
		type: paymentMethodType("type").notNull(),
		status: paymentMethodStatus("status").default("active").notNull(),
		isDefault: boolean("is_default").default(false),
		label: text("label"), // Ex: "Meu cartão principal", "PIX pessoal"

		// ===================================
		// PROVIDER (AGNÓSTICO)
		// ===================================
		provider: text("provider").notNull(), // "stripe" | "mercadopago"
		providerPaymentMethodId: text("provider_payment_method_id").unique(),
		providerCustomerId: text("provider_customer_id"),

		// ===================================
		// DADOS CARTÃO (type = "card")
		// Apenas dados não-sensíveis!
		// ===================================
		cardBrand: cardBrand("card_brand"),
		cardLast4: text("card_last4"), // Últimos 4 dígitos
		cardExpiryMonth: integer("card_expiry_month"), // 1-12
		cardExpiryYear: integer("card_expiry_year"), // Ex: 2027
		cardFingerprint: text("card_fingerprint"), // Hash único do cartão (Stripe)
		cardFunding: text("card_funding"), // "credit" | "debit" | "prepaid" | "unknown"
		cardCountry: text("card_country"), // Código do país (ex: "BR")

		// ===================================
		// DADOS PIX (type = "pix")
		// ===================================
		pixKey: text("pix_key"), // Chave PIX (se salva)
		pixKeyType: text("pix_key_type"), // "cpf" | "cnpj" | "email" | "phone" | "random"

		// ===================================
		// DADOS BOLETO (type = "boleto")
		// ===================================
		boletoTaxId: text("boleto_tax_id"), // CPF ou CNPJ do pagador

		// ===================================
		// DADOS CONTA BANCÁRIA (type = "bank_transfer")
		// ===================================
		bankName: text("bank_name"),
		bankCode: text("bank_code"), // Código do banco (ex: "001" para BB)
		bankAccountLast4: text("bank_account_last4"),
		bankAccountType: text("bank_account_type"), // "checking" | "savings"
		bankBranchCode: text("bank_branch_code"), // Agência

		// ===================================
		// DADOS PARA PAGAMENTO PENDENTE (PIX/BOLETO)
		// Usados durante o fluxo de pagamento único
		// ===================================
		// URL para completar o pagamento (QR code page, boleto PDF)
		nextActionUrl: text("next_action_url"),
		// URL direta do QR Code do PIX
		qrCodeUrl: text("qr_code_url"),
		// Código "copia e cola" do PIX
		qrCodeText: text("qr_code_text"),
		// URL do boleto (PDF ou página)
		barcodeUrl: text("barcode_url"),
		// Linha digitável do boleto
		barcodeDigitableLine: text("barcode_digitable_line"),
		// Data de expiração do PIX (24h) ou boleto (configurável)
		paymentExpiresAt: timestamp("payment_expires_at"),

		// ===================================
		// BILLING DETAILS (dados de cobrança)
		// ===================================
		billingDetails: jsonb("billing_details"), // {name, email, phone, address}

		// ===================================
		// CAPACIDADES DO MÉTODO
		// ===================================
		supportsRecurring: boolean("supports_recurring").default(false), // Suporta assinatura?
		supportsInstallments: boolean("supports_installments").default(false), // Suporta parcelamento?
		requiresAuthentication: boolean("requires_authentication").default(false), // Requer 3DS?
		maxInstallments: integer("max_installments"), // Máximo de parcelas permitidas

		// ===================================
		// METADADOS E RESTRIÇÕES
		// ===================================
		metadata: jsonb("metadata"), // Dados adicionais do provider
		usageRestrictions: jsonb("usage_restrictions"), // Restrições de uso
		supportedCurrencies: jsonb("supported_currencies"), // ["BRL", "USD"]

		// ===================================
		// TIMESTAMPS
		// ===================================
		lastUsedAt: timestamp("last_used_at"),
		verifiedAt: timestamp("verified_at"), // Quando foi verificado
		expiresAt: timestamp("expires_at"), // Expiração do cartão

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		// Índices para queries comuns
		index("payment_methods_user_id_idx").on(t.userId),
		index("payment_methods_type_idx").on(t.type),
		index("payment_methods_status_idx").on(t.status),
		index("payment_methods_is_default_idx").on(t.isDefault),
		index("payment_methods_provider_idx").on(t.provider),
		index("payment_methods_provider_pm_id_idx").on(t.providerPaymentMethodId),
		index("payment_methods_expires_at_idx").on(t.expiresAt),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/subscriptions.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import {
	billingCycle,
	collectionMethod,
	subscriptionStatus,
} from "../enums";
import { users } from "../users/users";
import { plans } from "./plans";
import { paymentMethods } from "./payment-methods";

/**
 * User Subscriptions - Assinaturas dos usuários
 *
 * Representa a relação entre usuário e plano com todos os detalhes
 * de cobrança, períodos e status.
 *
 * FLUXO DE STATUS (Stripe):
 * - incomplete → incomplete_expired (se não pagar no tempo)
 * - incomplete → active (pagamento bem-sucedido)
 * - trialing → active (trial terminou, pagamento OK)
 * - trialing → past_due (trial terminou, pagamento falhou)
 * - active → past_due (pagamento falhou)
 * - past_due → active (retry bem-sucedido)
 * - past_due → unpaid (após todas tentativas)
 * - past_due → canceled (cancelamento por inadimplência)
 * - active → canceled (cancelamento voluntário)
 * - active → paused (pausado pelo usuário/admin)
 * - paused → active (retomado)
 */
export const userSubscriptions = pgTable(
	"user_subscriptions",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		// ===================================
		// RELACIONAMENTOS
		// ===================================
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		planId: uuid("plan_id")
			.notNull()
			.references(() => plans.id),

		// ===================================
		// STATUS E CONTROLE
		// ===================================
		status: subscriptionStatus("status").default("incomplete").notNull(),

		// ===================================
		// DETALHES DE PREÇO E CICLO
		// ===================================
		billingCycle: billingCycle("billing_cycle"), // "monthly" | "yearly" | etc
		amount: integer("amount"), // Valor em centavos
		currency: text("currency").default("BRL").notNull(),

		// ===================================
		// PERÍODO DE COBRANÇA ATUAL
		// Crítico para saber quando cobrar novamente
		// ===================================
		currentPeriodStart: timestamp("current_period_start").notNull(),
		currentPeriodEnd: timestamp("current_period_end").notNull(),

		// ===================================
		// PERÍODO DE TRIAL
		// ===================================
		trialStart: timestamp("trial_start"),
		trialEnd: timestamp("trial_end"),

		// ===================================
		// DATAS GERAIS DA ASSINATURA
		// ===================================
		startsAt: timestamp("starts_at").notNull(), // Quando a assinatura começou
		endsAt: timestamp("ends_at"), // Quando a assinatura termina (se cancelada)
		nextBillingDate: timestamp("next_billing_date"), // Próxima cobrança

		// ===================================
		// CANCELAMENTO
		// ===================================
		cancelAtPeriodEnd: boolean("cancel_at_period_end").default(false), // Cancelar no fim do período?
		canceledAt: timestamp("canceled_at"), // Quando foi cancelado
		cancelReason: text("cancel_reason"), // Motivo do cancelamento
		cancelFeedback: text("cancel_feedback"), // Feedback do usuário

		// ===================================
		// PAUSA
		// ===================================
		pausedAt: timestamp("paused_at"),
		resumesAt: timestamp("resumes_at"), // Quando retomará automaticamente
		pauseReason: text("pause_reason"),

		// ===================================
		// DETALHES DE PAGAMENTO
		// ===================================
		defaultPaymentMethodId: uuid("default_payment_method_id").references(
			() => paymentMethods.id,
			{ onDelete: "set null" },
		),
		collectionMethod: collectionMethod("collection_method").default(
			"charge_automatically",
		),
		daysUntilDue: integer("days_until_due"), // Para send_invoice

		// ===================================
		// REFERÊNCIAS DO PROVIDER
		// ===================================
		latestInvoiceId: text("latest_invoice_id"), // ID da última fatura no provider
		pendingUpdateId: text("pending_update_id"), // ID de mudança pendente

		// ===================================
		// PROVIDER (AGNÓSTICO)
		// ===================================
		provider: text("provider").notNull(), // "stripe" | "mercadopago"
		providerSubscriptionId: text("provider_subscription_id").unique(),
		providerCustomerId: text("provider_customer_id"),
		providerPriceId: text("provider_price_id"), // ID do preço no provider

		// ===================================
		// METADADOS
		// ===================================
		metadata: jsonb("metadata"), // Dados adicionais

		// ===================================
		// TIMESTAMPS
		// ===================================
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		// Índices para queries comuns
		index("user_subscriptions_user_id_idx").on(t.userId),
		index("user_subscriptions_plan_id_idx").on(t.planId),
		index("user_subscriptions_status_idx").on(t.status),
		index("user_subscriptions_billing_cycle_idx").on(t.billingCycle),
		index("user_subscriptions_current_period_end_idx").on(t.currentPeriodEnd),
		index("user_subscriptions_next_billing_date_idx").on(t.nextBillingDate),
		index("user_subscriptions_cancel_at_period_end_idx").on(t.cancelAtPeriodEnd),
		index("user_subscriptions_provider_idx").on(t.provider),
		index("user_subscriptions_provider_sub_id_idx").on(t.providerSubscriptionId),
		index("user_subscriptions_provider_customer_id_idx").on(t.providerCustomerId),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/transactions.ts =====
import {
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import {
	paymentMethodType,
	transactionStatus,
	transactionType,
} from "../enums";
import { users } from "../users/users";
import { paymentMethods } from "./payment-methods";
import { userSubscriptions } from "./subscriptions";

/**
 * Transactions - Histórico completo de transações financeiras
 *
 * Esta tabela serve como:
 * 1. Registro de auditoria de todas as operações financeiras
 * 2. Reconciliação com o provedor de pagamentos (Stripe)
 * 3. Base para relatórios financeiros
 *
 * IMPORTANTE:
 * - NUNCA deletar transações (auditoria)
 * - Valores sempre em centavos (evitar float)
 * - Guardar IDs do provider para reconciliação
 *
 * TIPOS DE TRANSAÇÃO:
 * - charge: Cobrança (assinatura ou avulso)
 * - refund: Reembolso total ou parcial
 * - payment: Pagamento genérico
 * - invoice_payment: Pagamento de fatura
 * - adjustment: Ajuste manual
 * - fee: Taxa cobrada
 * - dispute: Disputa/chargeback
 * - transfer: Transferência
 * - payout: Saque para conta bancária
 */
export const transactions = pgTable(
	"transactions",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		// ===================================
		// RELACIONAMENTOS
		// ===================================
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id), // Denormalized for fast queries
		subscriptionId: uuid("subscription_id").references(
			() => userSubscriptions.id,
			{ onDelete: "set null" },
		),
		paymentMethodId: uuid("payment_method_id").references(
			() => paymentMethods.id,
			{ onDelete: "set null" },
		),
		// Para reembolsos, referência à transação original
		originalTransactionId: uuid("original_transaction_id"),

		// ===================================
		// TIPO E STATUS
		// ===================================
		type: transactionType("type").notNull(),
		status: transactionStatus("status").default("pending").notNull(),

		// ===================================
		// VALORES FINANCEIROS (em centavos)
		// ===================================
		amount: integer("amount").notNull(), // Valor da transação
		amountGross: integer("amount_gross"), // Valor bruto (antes de taxas)
		amountFees: integer("amount_fees"), // Taxas cobradas pelo provider
		amountNet: integer("amount_net"), // Valor líquido (após taxas)
		amountRefunded: integer("amount_refunded").default(0), // Valor já reembolsado
		currency: text("currency").default("BRL").notNull(),

		// ===================================
		// MÉTODO DE PAGAMENTO USADO
		// ===================================
		paymentMethod: paymentMethodType("payment_method"),

		// ===================================
		// DESCRIÇÃO E IDENTIFICAÇÃO
		// ===================================
		description: text("description"), // Descrição da transação
		statementDescriptor: text("statement_descriptor"), // O que aparece no extrato
		receiptNumber: text("receipt_number"), // Número do recibo
		receiptUrl: text("receipt_url"), // URL do comprovante

		// ===================================
		// IDS DO PROVIDER (STRIPE)
		// ===================================
		provider: text("provider").notNull(), // "stripe" | "mercadopago"
		providerTransactionId: text("provider_transaction_id").unique(), // ID único no provider
		providerPaymentIntentId: text("provider_payment_intent_id"), // PaymentIntent ID
		providerChargeId: text("provider_charge_id"), // Charge ID
		providerBalanceTransactionId: text("provider_balance_transaction_id"), // Balance Transaction ID
		providerInvoiceId: text("provider_invoice_id"), // Invoice ID (se houver)
		providerRefundId: text("provider_refund_id"), // Refund ID (se for reembolso)
		providerDisputeId: text("provider_dispute_id"), // Dispute ID (se houver)

		// ===================================
		// DETALHES DE ERRO
		// ===================================
		errorCode: text("error_code"), // Código do erro (decline_code, etc)
		errorMessage: text("error_message"), // Mensagem de erro
		errorDeclineCode: text("error_decline_code"), // Código de recusa do cartão
		errorType: text("error_type"), // Tipo de erro (card_error, api_error, etc)

		// ===================================
		// DETALHES DE DISPUTA (chargeback)
		// ===================================
		disputeReason: text("dispute_reason"), // Motivo da disputa
		disputeStatus: text("dispute_status"), // Status da disputa
		disputeAmount: integer("dispute_amount"), // Valor em disputa

		// ===================================
		// DETALHES DE REEMBOLSO
		// ===================================
		refundReason: text("refund_reason"), // Motivo do reembolso
		refundedBy: uuid("refunded_by"), // Quem autorizou o reembolso

		// ===================================
		// METADADOS
		// ===================================
		metadata: jsonb("metadata"), // Dados adicionais do provider
		internalNotes: text("internal_notes"), // Notas internas (não visível ao usuário)

		// ===================================
		// INFORMAÇÕES DE IP/LOCALIZAÇÃO (auditoria)
		// ===================================
		ipAddress: text("ip_address"), // IP do cliente
		userAgent: text("user_agent"), // User agent do browser
		country: text("country"), // País do cartão/cliente

		// ===================================
		// TIMESTAMPS
		// ===================================
		processedAt: timestamp("processed_at"), // Quando foi processado pelo provider
		completedAt: timestamp("completed_at"), // Quando foi concluído
		failedAt: timestamp("failed_at"), // Quando falhou
		refundedAt: timestamp("refunded_at"), // Quando foi reembolsado
		disputedAt: timestamp("disputed_at"), // Quando entrou em disputa

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		// Índices para queries comuns
		index("transactions_user_id_idx").on(t.userId),
		index("transactions_subscription_id_idx").on(t.subscriptionId),
		index("transactions_payment_method_id_idx").on(t.paymentMethodId),
		index("transactions_original_tx_id_idx").on(t.originalTransactionId),
		index("transactions_type_idx").on(t.type),
		index("transactions_status_idx").on(t.status),
		index("transactions_payment_method_type_idx").on(t.paymentMethod),
		index("transactions_provider_idx").on(t.provider),
		index("transactions_provider_tx_id_idx").on(t.providerTransactionId),
		index("transactions_provider_pi_id_idx").on(t.providerPaymentIntentId),
		index("transactions_provider_charge_id_idx").on(t.providerChargeId),
		index("transactions_provider_invoice_id_idx").on(t.providerInvoiceId),
		index("transactions_created_at_idx").on(t.createdAt),
		index("transactions_completed_at_idx").on(t.completedAt),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/plans.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { subscriptionPlan, userRoles } from "../enums";

/**
 * Plans - Planos de assinatura disponíveis
 *
 * Define os produtos/planos que podem ser adquiridos.
 * Cada plano tem preços para diferentes ciclos (mensal, anual)
 * e pode ter um período de trial.
 *
 * IMPORTANTE:
 * - Valores sempre em centavos (evitar float)
 * - providerProductId é o ID do Product no Stripe
 * - providerPriceMonthlyId/YearlyId são os IDs dos Prices no Stripe
 */
export const plans = pgTable(
	"plans",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		// ===================================
		// IDENTIFICAÇÃO
		// ===================================
		type: subscriptionPlan("type").unique().notNull(),
		name: text("name").notNull(),
		description: text("description"),

		// ===================================
		// RESTRIÇÕES POR ROLE
		// ===================================
		allowedRole: userRoles("allowed_role").notNull(),

		// ===================================
		// VALORES (em centavos)
		// ===================================
		priceMonthly: integer("price_monthly"), // Preço mensal em centavos
		priceYearly: integer("price_yearly"), // Preço anual em centavos
		currency: text("currency").default("BRL").notNull(),
		billingCycle: text("billing_cycle"), // "monthly" | "yearly" | null (free)

		// ===================================
		// TRIAL (período de teste)
		// ===================================
		trialDays: integer("trial_days"), // Dias de trial (ex: 7, 14, 30)
		hasFreeTrial: boolean("has_free_trial").default(false),

		// ===================================
		// PROVIDER IDS (AGNÓSTICO)
		// ===================================
		provider: text("provider").default("stripe").notNull(),
		providerProductId: text("provider_product_id"), // Product ID no Stripe
		providerPriceMonthlyId: text("provider_price_monthly_id"), // Price ID mensal
		providerPriceYearlyId: text("provider_price_yearly_id"), // Price ID anual

		// ===================================
		// VERSIONAMENTO E CONTROLE
		// ===================================
		version: integer("version").default(1).notNull(), // Para controle de versão
		isActive: boolean("is_active").default(true), // Plano ativo para venda?
		isPublic: boolean("is_public").default(true), // Visível publicamente?
		isDefault: boolean("is_default").default(false), // Plano padrão para o role?

		// ===================================
		// LIMITES E CONFIGURAÇÕES
		// ===================================
		sortOrder: integer("sort_order").default(0), // Ordem de exibição
		maxUsers: integer("max_users"), // Máximo de usuários (para enterprise)

		// ===================================
		// METADADOS
		// ===================================
		metadata: jsonb("metadata"), // Dados adicionais
		highlightFeatures: jsonb("highlight_features"), // Features para destaque na UI

		// ===================================
		// TIMESTAMPS
		// ===================================
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		index("plans_type_idx").on(t.type),
		index("plans_allowed_role_idx").on(t.allowedRole),
		index("plans_is_active_idx").on(t.isActive),
		index("plans_is_public_idx").on(t.isPublic),
		index("plans_is_default_idx").on(t.isDefault),
		index("plans_provider_idx").on(t.provider),
		index("plans_provider_product_id_idx").on(t.providerProductId),
		index("plans_sort_order_idx").on(t.sortOrder),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/webhook-logs.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";

/**
 * Webhook Logs - Registro de todos os webhooks recebidos
 *
 * Esta tabela serve para:
 * 1. Garantir idempotência (mesmo evento não é processado duas vezes)
 * 2. Auditoria de todos os eventos recebidos
 * 3. Reprocessamento em caso de falha
 * 4. Debug e troubleshooting
 *
 * IMPORTANTE:
 * - eventId deve ser ÚNICO (garantido pelo provider)
 * - payload é armazenado completo em JSONB
 * - Funciona para qualquer provider (Stripe, Mux, MercadoPago, etc)
 *
 * EVENTOS CRÍTICOS (Stripe):
 * - customer.subscription.created/updated/deleted
 * - invoice.payment_succeeded/failed
 * - charge.succeeded (PIX/Boleto)
 * - payment_intent.succeeded/failed
 * - charge.dispute.created
 */
export const webhookLogs = pgTable(
	"webhook_logs",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		// ===================================
		// IDENTIFICAÇÃO DO EVENTO
		// ===================================
		provider: text("provider").notNull(), // "stripe" | "mux" | "mercadopago"
		eventType: text("event_type").notNull(), // "customer.subscription.updated"
		eventId: text("event_id").notNull().unique(), // ID único do provider (idempotency)

		// ===================================
		// PAYLOAD COMPLETO
		// ===================================
		payload: jsonb("payload").notNull(), // Payload completo do webhook

		// ===================================
		// PROCESSAMENTO
		// ===================================
		processed: boolean("processed").default(false),
		processingAttempts: integer("processing_attempts").default(0), // Quantas vezes tentou processar
		maxAttempts: integer("max_attempts").default(5), // Máximo de tentativas

		// ===================================
		// RESULTADO DO PROCESSAMENTO
		// ===================================
		processedResult: text("processed_result"), // "success" | "failed" | "skipped"
		processedData: jsonb("processed_data"), // Dados resultantes do processamento

		// ===================================
		// ERROS
		// ===================================
		error: text("error"), // Mensagem de erro principal
		errorCode: text("error_code"), // Código do erro
		errorStack: text("error_stack"), // Stack trace (para debug)
		lastError: text("last_error"), // Último erro (se múltiplas tentativas)

		// ===================================
		// VERIFICAÇÃO DE ASSINATURA
		// ===================================
		signatureHeader: text("signature_header"), // Header de assinatura recebido
		signatureVerified: boolean("signature_verified").default(false), // Se passou na verificação

		// ===================================
		// INFORMAÇÕES DE ORIGEM
		// ===================================
		ipAddress: text("ip_address"), // IP que enviou o webhook
		userAgent: text("user_agent"), // User-Agent do request

		// ===================================
		// RELACIONAMENTOS (opcionais)
		// Populados após processamento
		// ===================================
		relatedUserId: uuid("related_user_id"), // Usuário relacionado ao evento
		relatedSubscriptionId: uuid("related_subscription_id"), // Assinatura relacionada
		relatedTransactionId: uuid("related_transaction_id"), // Transação criada/atualizada

		// ===================================
		// TIMESTAMPS
		// ===================================
		receivedAt: timestamp("received_at")
			.$defaultFn(() => new Date())
			.notNull(), // Quando foi recebido
		processedAt: timestamp("processed_at"), // Quando foi processado com sucesso
		lastAttemptAt: timestamp("last_attempt_at"), // Última tentativa de processamento
		nextAttemptAt: timestamp("next_attempt_at"), // Próxima tentativa agendada

		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		// Índices para queries comuns
		index("webhook_logs_provider_idx").on(t.provider),
		index("webhook_logs_event_type_idx").on(t.eventType),
		index("webhook_logs_event_id_idx").on(t.eventId),
		index("webhook_logs_processed_idx").on(t.processed),
		index("webhook_logs_processed_result_idx").on(t.processedResult),
		index("webhook_logs_related_user_id_idx").on(t.relatedUserId),
		index("webhook_logs_related_sub_id_idx").on(t.relatedSubscriptionId),
		index("webhook_logs_received_at_idx").on(t.receivedAt),
		index("webhook_logs_next_attempt_at_idx").on(t.nextAttemptAt),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/invoices.ts =====
import {
	boolean,
	index,
	integer,
	jsonb,
	pgTable,
	text,
	timestamp,
	uuid,
} from "drizzle-orm/pg-core";
import { v7 as uuidv7 } from "uuid";
import { collectionMethod, invoiceStatus } from "../enums";
import { users } from "../users/users";
import { userSubscriptions } from "./subscriptions";

/**
 * Invoices - Faturas de cobrança
 *
 * Representa faturas geradas para usuários, seja por assinatura
 * ou cobranças avulsas. Alinhado com Stripe Invoice API.
 *
 * BILLING REASONS (Stripe):
 * - subscription_create: Primeira fatura da assinatura
 * - subscription_cycle: Cobrança recorrente
 * - subscription_update: Alteração no meio do ciclo (proration)
 * - subscription_threshold: Limite de uso atingido
 * - manual: Criada manualmente
 *
 * FLUXO DE STATUS:
 * - draft → open (quando finalizada)
 * - open → paid (pagamento bem-sucedido)
 * - open → void (cancelada)
 * - open → uncollectible (após tentativas)
 */
export const invoices = pgTable(
	"invoices",
	{
		id: uuid("id")
			.primaryKey()
			.$defaultFn(() => uuidv7()),

		// ===================================
		// RELACIONAMENTOS
		// ===================================
		userId: uuid("user_id")
			.notNull()
			.references(() => users.id, { onDelete: "cascade" }),
		subscriptionId: uuid("subscription_id").references(
			() => userSubscriptions.id,
			{ onDelete: "set null" },
		),

		// ===================================
		// IDENTIFICAÇÃO
		// ===================================
		number: text("number").unique(), // Número da fatura (INV-2024-0001)
		status: invoiceStatus("status").default("draft").notNull(),

		// ===================================
		// VALORES (em centavos)
		// ===================================
		subtotal: integer("subtotal").notNull(), // Subtotal (antes de descontos/taxas)
		discount: integer("discount").default(0), // Desconto aplicado
		tax: integer("tax").default(0), // Impostos
		amountDue: integer("amount_due").notNull(), // Valor a pagar
		amountPaid: integer("amount_paid").default(0), // Valor já pago
		amountRemaining: integer("amount_remaining").notNull(), // Valor restante
		currency: text("currency").default("BRL").notNull(),

		// ===================================
		// PERÍODO DE COBRANÇA
		// ===================================
		periodStart: timestamp("period_start"), // Início do período cobrado
		periodEnd: timestamp("period_end"), // Fim do período cobrado

		// ===================================
		// DATAS
		// ===================================
		dueDate: timestamp("due_date"), // Data de vencimento
		paidAt: timestamp("paid_at"), // Quando foi pago
		voidedAt: timestamp("voided_at"), // Quando foi anulada
		finalizedAt: timestamp("finalized_at"), // Quando saiu de draft

		// ===================================
		// COBRANÇA
		// ===================================
		billingReason: text("billing_reason"), // subscription_create, subscription_cycle, manual, etc
		collectionMethod: collectionMethod("collection_method").default(
			"charge_automatically",
		),
		attemptCount: integer("attempt_count").default(0), // Tentativas de cobrança
		nextPaymentAttempt: timestamp("next_payment_attempt"), // Próxima tentativa

		// ===================================
		// DESCONTOS
		// ===================================
		discountCode: text("discount_code"), // Código do cupom aplicado
		discountReason: text("discount_reason"), // Motivo do desconto

		// ===================================
		// DOCUMENTOS
		// ===================================
		hostedInvoiceUrl: text("hosted_invoice_url"), // URL para cliente pagar
		invoicePdf: text("invoice_pdf"), // URL do PDF da fatura
		receiptNumber: text("receipt_number"), // Número do recibo

		// ===================================
		// DESCRIÇÃO
		// ===================================
		description: text("description"), // Descrição da fatura
		footer: text("footer"), // Texto no rodapé da fatura
		memo: text("memo"), // Memo interno

		// ===================================
		// CLIENTE (snapshot no momento da fatura)
		// ===================================
		customerEmail: text("customer_email"), // Email no momento da fatura
		customerName: text("customer_name"), // Nome no momento da fatura
		customerTaxId: text("customer_tax_id"), // CPF/CNPJ no momento

		// ===================================
		// PROVIDER (AGNÓSTICO)
		// ===================================
		provider: text("provider").notNull(), // "stripe" | "mercadopago"
		providerInvoiceId: text("provider_invoice_id").unique(), // ID da fatura no provider
		providerCustomerId: text("provider_customer_id"), // ID do cliente no provider
		providerSubscriptionId: text("provider_subscription_id"), // ID da assinatura no provider
		providerPaymentIntentId: text("provider_payment_intent_id"), // PaymentIntent associado
		providerChargeId: text("provider_charge_id"), // Charge associado

		// ===================================
		// ITENS DA FATURA
		// Armazenado como JSONB para flexibilidade
		// ===================================
		lines: jsonb("lines"), // Array de {description, amount, quantity, period}

		// ===================================
		// METADADOS
		// ===================================
		metadata: jsonb("metadata"), // Dados adicionais

		// ===================================
		// TIMESTAMPS
		// ===================================
		createdAt: timestamp("created_at")
			.$defaultFn(() => new Date())
			.notNull(),
		updatedAt: timestamp("updated_at")
			.$defaultFn(() => new Date())
			.$onUpdate(() => new Date())
			.notNull(),
	},
	(t) => [
		// Índices para queries comuns
		index("invoices_user_id_idx").on(t.userId),
		index("invoices_subscription_id_idx").on(t.subscriptionId),
		index("invoices_number_idx").on(t.number),
		index("invoices_status_idx").on(t.status),
		index("invoices_due_date_idx").on(t.dueDate),
		index("invoices_paid_at_idx").on(t.paidAt),
		index("invoices_billing_reason_idx").on(t.billingReason),
		index("invoices_provider_idx").on(t.provider),
		index("invoices_provider_invoice_id_idx").on(t.providerInvoiceId),
		index("invoices_created_at_idx").on(t.createdAt),
	],
);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/billing/index.ts =====
export * from "./plans";
export * from "./plan-features";
export * from "./permissions";
export * from "./subscriptions";
export * from "./transactions";
export * from "./payment-methods";
export * from "./invoices";
export * from "./webhook-logs";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/index.ts =====
export * from "./auth";
export * from "./billing";
export * from "./categories";
export * from "./profiles";
export * from "./users";
export * from "./videos";
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/enums.ts =====
import { pgEnum } from "drizzle-orm/pg-core";

// ===================================
// USER ENUMS
// ===================================
export const userRoles = pgEnum("user_roles", [
	"none",
	"resident",
	"syndic",
	"enterprise",
	"marketing",
	"local_company",
	"admin",
]);

// ===================================
// PROFILE STATUS ENUM (UNIFICADO)
// ===================================
// Responsabilidade: Completude do perfil (não confundir com status de assinatura)
export const profileStatus = pgEnum("profile_status", [
	"incomplete", // Perfil em criação - campos obrigatórios ainda não preenchidos
	"complete", // Perfil completo - todos os campos obrigatórios preenchidos
	"inactive", // Perfil desativado - usuário desativou ou foi banido
]);

// ===================================
// SYNDIC ENUMS
// ===================================
export const syndicType = pgEnum("syndic_type", ["morador", "profissional"]);

// ===================================
// VIDEO ENUMS
// ===================================
export const videoType = pgEnum("video_type", [
	"instrucional",
	"institucional",
	"curriculo",
	"sindico",
	"curso",
]);

export const videoStatus = pgEnum("video_status", [
	"pending",
	"processing",
	"ready",
	"failed",
]);

// ===================================
// SUBSCRIPTION ENUMS
// ===================================
// Status da assinatura (alinhado com Stripe)
export const subscriptionStatus = pgEnum("subscription_status", [
	"incomplete", // Pagamento inicial não completado
	"incomplete_expired", // Expirou antes de completar
	"trialing", // Em período de teste
	"active", // Ativa e em dia
	"past_due", // Pagamento atrasado (retry em andamento)
	"canceled", // Cancelada
	"unpaid", // Não paga após várias tentativas
	"paused", // Temporariamente pausada
]);

export const subscriptionPlan = pgEnum("subscription_plan", [
	"resident_base",
	"resident_paid",
	"syndic_n1",
	"syndic_n2",
	"syndic_n3",
	"enterprise_plus",
	"enterprise_pro",
	"marketing_standard",
	"local_company_standard",
]);

// ===================================
// BILLING ENUMS
// ===================================

// Tipos de método de pagamento (alinhado com Stripe)
export const paymentMethodType = pgEnum("payment_method_type", [
	"card", // Cartão de crédito/débito (Stripe unifica)
	"pix", // PIX (Brasil - pagamento único)
	"boleto", // Boleto bancário (Brasil - pagamento único)
	"bank_transfer", // Transferência bancária
	"google_pay", // Google Pay (wallet)
	"apple_pay", // Apple Pay (wallet)
]);

// Bandeiras de cartão
export const cardBrand = pgEnum("card_brand", [
	"visa",
	"mastercard",
	"amex",
	"elo",
	"hipercard",
	"discover",
	"diners",
	"jcb",
	"unionpay",
	"unknown",
]);

// Status do método de pagamento
export const paymentMethodStatus = pgEnum("payment_method_status", [
	"active", // Pode ser usado
	"inactive", // Desativado pelo usuário
	"expired", // Cartão expirado
	"requires_action", // Precisa de ação (ex: 3DS)
	"failed", // Falhou na verificação
]);

// Tipos de transação (alinhado com Stripe Balance Transaction)
export const transactionType = pgEnum("transaction_type", [
	"charge", // Cobrança
	"refund", // Reembolso
	"payment", // Pagamento genérico
	"invoice_payment", // Pagamento de fatura
	"adjustment", // Ajuste manual
	"fee", // Taxa
	"dispute", // Disputa/chargeback
	"transfer", // Transferência
	"payout", // Saque para conta bancária
]);

// Status da transação (alinhado com Stripe)
export const transactionStatus = pgEnum("transaction_status", [
	"pending", // Pendente (aguardando processamento)
	"processing", // Em processamento
	"requires_action", // Requer ação (ex: 3DS, PIX pendente)
	"succeeded", // Bem-sucedido
	"failed", // Falhou
	"canceled", // Cancelado
	"refunded", // Totalmente reembolsado
	"partially_refunded", // Parcialmente reembolsado
	"disputed", // Em disputa
	"void", // Anulado
]);

// Ciclo de cobrança
export const billingCycle = pgEnum("billing_cycle", [
	"monthly", // Mensal
	"quarterly", // Trimestral
	"semiannual", // Semestral
	"yearly", // Anual
]);

// Método de coleta de pagamento
export const collectionMethod = pgEnum("collection_method", [
	"charge_automatically", // Cobrar automaticamente
	"send_invoice", // Enviar fatura
]);

// ===================================
// TAX ID ENUMS (Stripe-compatible)
// ===================================
// Tipos de documento fiscal para integração com Stripe
// Referência: https://stripe.com/docs/api/customer_tax_ids/object
export const taxIdType = pgEnum("tax_id_type", [
	"br_cpf", // CPF - Cadastro de Pessoa Física (Brasil)
	"br_cnpj", // CNPJ - Cadastro Nacional de Pessoa Jurídica (Brasil)
]);

// ===================================
// INVOICE ENUMS
// ===================================
// Status da fatura (alinhado com Stripe)
export const invoiceStatus = pgEnum("invoice_status", [
	"draft", // Rascunho
	"open", // Aberta (aguardando pagamento)
	"paid", // Paga
	"void", // Anulada
	"uncollectible", // Incobrável
]);

// Razão do desconto na fatura
export const invoiceDiscountReason = pgEnum("invoice_discount_reason", [
	"coupon", // Cupom de desconto
	"promotion", // Promoção
	"loyalty", // Fidelidade
	"manual", // Ajuste manual
]);
\n\n===== FILE: ./src/infrastructure/database/drizzle/schema/relations.ts =====
import { defineRelations } from "drizzle-orm";
import * as schema from "./index";

export const relations = defineRelations(schema, (r) => ({
	// ===================================
	// USERS → PROFILES (Polymorphic)
	// ===================================
	users: {
		// Perfis (um user tem no máximo um de cada, baseado no role)
		residentProfile: r.one.residentProfile({
			from: r.users.id,
			to: r.residentProfile.userId,
		}),
		syndicProfile: r.one.syndicProfile({
			from: r.users.id,
			to: r.syndicProfile.userId,
		}),
		enterpriseProfile: r.one.enterpriseProfile({
			from: r.users.id,
			to: r.enterpriseProfile.userId,
		}),
		localCompanyProfile: r.one.localCompanyProfile({
			from: r.users.id,
			to: r.localCompanyProfile.userId,
		}),
		marketingProfile: r.one.marketingProfile({
			from: r.users.id,
			to: r.marketingProfile.userId,
		}),

		// Vídeos do usuário
		videos: r.many.videos({
			from: r.users.id,
			to: r.videos.authorUserId,
		}),

		// Sessions e Accounts
		sessions: r.many.sessions({
			from: r.users.id,
			to: r.sessions.userId,
		}),
		accounts: r.many.accounts({
			from: r.users.id,
			to: r.accounts.userId,
		}),

		// Billing
		subscriptions: r.many.userSubscriptions({
			from: r.users.id,
			to: r.userSubscriptions.userId,
		}),
		transactions: r.many.transactions({
			from: r.users.id,
			to: r.transactions.userId,
		}),
		paymentMethods: r.many.paymentMethods({
			from: r.users.id,
			to: r.paymentMethods.userId,
		}),
		invoices: r.many.invoices({
			from: r.users.id,
			to: r.invoices.userId,
		}),
	},

	// ===================================
	// PROFILES → USER
	// ===================================
	residentProfile: {
		user: r.one.users({
			from: r.residentProfile.userId,
			to: r.users.id,
		}),
	},

	syndicProfile: {
		user: r.one.users({
			from: r.syndicProfile.userId,
			to: r.users.id,
		}),
	},

	enterpriseProfile: {
		user: r.one.users({
			from: r.enterpriseProfile.userId,
			to: r.users.id,
		}),
		// Categorias (via junction table)
		categories: r.many.serviceCategories({
			from: r.enterpriseProfile.id.through(
				r.enterpriseProfileCategories.profileId,
			),
			to: r.serviceCategories.id.through(
				r.enterpriseProfileCategories.categoryId,
			),
		}),
		// Subcategorias (via junction table)
		subcategories: r.many.serviceCategories({
			from: r.enterpriseProfile.id.through(
				r.enterpriseProfileSubcategories.profileId,
			),
			to: r.serviceCategories.id.through(
				r.enterpriseProfileSubcategories.subcategoryId,
			),
		}),
	},

	localCompanyProfile: {
		user: r.one.users({
			from: r.localCompanyProfile.userId,
			to: r.users.id,
		}),
	},

	marketingProfile: {
		user: r.one.users({
			from: r.marketingProfile.userId,
			to: r.users.id,
		}),
	},

	// ===================================
	// VIDEOS
	// ===================================
	videos: {
		author: r.one.users({
			from: r.videos.authorUserId,
			to: r.users.id,
		}),
		category: r.one.serviceCategories({
			from: r.videos.serviceCategoryId,
			to: r.serviceCategories.id,
		}),
		views: r.many.videoViews({
			from: r.videos.id,
			to: r.videoViews.videoId,
		}),
		likes: r.many.videoLikes({
			from: r.videos.id,
			to: r.videoLikes.videoId,
		}),
		comments: r.many.videoComments({
			from: r.videos.id,
			to: r.videoComments.videoId,
		}),
	},

	videoViews: {
		video: r.one.videos({
			from: r.videoViews.videoId,
			to: r.videos.id,
		}),
		user: r.one.users({
			from: r.videoViews.userId,
			to: r.users.id,
		}),
		session: r.one.sessions({
			from: r.videoViews.sessionId,
			to: r.sessions.id,
		}),
	},

	videoLikes: {
		video: r.one.videos({
			from: r.videoLikes.videoId,
			to: r.videos.id,
		}),
		user: r.one.users({
			from: r.videoLikes.userId,
			to: r.users.id,
		}),
	},

	videoComments: {
		video: r.one.videos({
			from: r.videoComments.videoId,
			to: r.videos.id,
		}),
		user: r.one.users({
			from: r.videoComments.userId,
			to: r.users.id,
		}),
	},

	// ===================================
	// CATEGORIES
	// ===================================
	serviceCategories: {
		// Self-reference para hierarquia
		parent: r.one.serviceCategories({
			from: r.serviceCategories.parentId,
			to: r.serviceCategories.id,
			alias: "parent",
		}),
		children: r.many.serviceCategories({
			from: r.serviceCategories.id,
			to: r.serviceCategories.parentId,
			alias: "children",
		}),
		// Vídeos da categoria
		videos: r.many.videos({
			from: r.serviceCategories.id,
			to: r.videos.serviceCategoryId,
		}),
	},

	// ===================================
	// AUTH
	// ===================================
	sessions: {
		user: r.one.users({
			from: r.sessions.userId,
			to: r.users.id,
		}),
	},

	accounts: {
		user: r.one.users({
			from: r.accounts.userId,
			to: r.users.id,
		}),
	},

	// ===================================
	// BILLING - PLANS
	// ===================================
	plans: {
		features: r.many.planFeatures({
			from: r.plans.id,
			to: r.planFeatures.planId,
		}),
		permissions: r.many.planPermissions({
			from: r.plans.id,
			to: r.planPermissions.planId,
		}),
		subscriptions: r.many.userSubscriptions({
			from: r.plans.id,
			to: r.userSubscriptions.planId,
		}),
	},

	planFeatures: {
		plan: r.one.plans({
			from: r.planFeatures.planId,
			to: r.plans.id,
		}),
	},

	permissions: {
		planPermissions: r.many.planPermissions({
			from: r.permissions.id,
			to: r.planPermissions.permissionId,
		}),
	},

	planPermissions: {
		plan: r.one.plans({
			from: r.planPermissions.planId,
			to: r.plans.id,
		}),
		permission: r.one.permissions({
			from: r.planPermissions.permissionId,
			to: r.permissions.id,
		}),
	},

	// ===================================
	// BILLING - SUBSCRIPTIONS
	// ===================================
	userSubscriptions: {
		user: r.one.users({
			from: r.userSubscriptions.userId,
			to: r.users.id,
		}),
		plan: r.one.plans({
			from: r.userSubscriptions.planId,
			to: r.plans.id,
		}),
		defaultPaymentMethod: r.one.paymentMethods({
			from: r.userSubscriptions.defaultPaymentMethodId,
			to: r.paymentMethods.id,
		}),
		transactions: r.many.transactions({
			from: r.userSubscriptions.id,
			to: r.transactions.subscriptionId,
		}),
		invoices: r.many.invoices({
			from: r.userSubscriptions.id,
			to: r.invoices.subscriptionId,
		}),
	},

	// ===================================
	// BILLING - TRANSACTIONS
	// ===================================
	transactions: {
		user: r.one.users({
			from: r.transactions.userId,
			to: r.users.id,
		}),
		subscription: r.one.userSubscriptions({
			from: r.transactions.subscriptionId,
			to: r.userSubscriptions.id,
		}),
		paymentMethodRef: r.one.paymentMethods({
			from: r.transactions.paymentMethodId,
			to: r.paymentMethods.id,
		}),
		// Transação original (para reembolsos)
		originalTransaction: r.one.transactions({
			from: r.transactions.originalTransactionId,
			to: r.transactions.id,
			alias: "originalTransaction",
		}),
		// Reembolsos desta transação
		refunds: r.many.transactions({
			from: r.transactions.id,
			to: r.transactions.originalTransactionId,
			alias: "refunds",
		}),
	},

	// ===================================
	// BILLING - PAYMENT METHODS
	// ===================================
	paymentMethods: {
		user: r.one.users({
			from: r.paymentMethods.userId,
			to: r.users.id,
		}),
		// Assinaturas que usam este método como padrão
		subscriptions: r.many.userSubscriptions({
			from: r.paymentMethods.id,
			to: r.userSubscriptions.defaultPaymentMethodId,
		}),
		// Transações feitas com este método
		transactions: r.many.transactions({
			from: r.paymentMethods.id,
			to: r.transactions.paymentMethodId,
		}),
	},

	// ===================================
	// BILLING - INVOICES
	// ===================================
	invoices: {
		user: r.one.users({
			from: r.invoices.userId,
			to: r.users.id,
		}),
		subscription: r.one.userSubscriptions({
			from: r.invoices.subscriptionId,
			to: r.userSubscriptions.id,
		}),
	},
}));
\n\n===== FILE: ./src/infrastructure/database/drizzle/unit-of-work/unit-of-work.ts =====
import { AsyncLocalStorage } from "node:async_hooks";
import type { Logger } from "pino";
import type { DrizzleClient } from "../client";
import type {
	DatabaseTransactionClient,
	IUnitOfWork,
} from "./unit-of-work.interface";

const transactionContext = new AsyncLocalStorage<DatabaseTransactionClient>();

interface DrizzleUnitOfWorkDependencies {
	database: DrizzleClient;
	logger: Logger;
}

export class DrizzleUnitOfWork implements IUnitOfWork {
	private readonly database: DrizzleClient;
	private readonly logger: Logger;

	constructor({ database, logger }: DrizzleUnitOfWorkDependencies) {
		this.database = database;
		this.logger = logger;
	}

	async run<T>(callback: () => Promise<T>): Promise<T> {
		const existingTx = transactionContext.getStore();
		if (existingTx) return callback();

		return this.database.transaction(async (tx) => {
			return transactionContext.run(tx, async () => {
				try {
					return await callback();
				} catch (error) {
					this.logger.error({ err: error }, "Transaction failed, rolling back");
					throw error;
				}
			});
		});
	}

	getTransactionClient(): DatabaseTransactionClient | DrizzleClient {
		const txClient = transactionContext.getStore();
		return txClient ?? this.database;
	}

	isInTransaction(): boolean {
		return transactionContext.getStore() !== undefined;
	}
}
\n\n===== FILE: ./src/infrastructure/database/drizzle/unit-of-work/unit-of-work.interface.ts =====
import type { DrizzleClient } from "../client";

/**
 * Tipo extraído do cliente de transação do Drizzle
 */
export type DatabaseTransactionClient = Parameters<
	Parameters<DrizzleClient["transaction"]>[0]
>[0];

export interface IUnitOfWork {
	/**
	 * ✅ Executa callback dentro de uma transação
	 *
	 * - Se já houver uma transação ativa, reutiliza (evita nested transactions)
	 * - Em caso de erro, faz rollback automático
	 * - Em caso de sucesso, faz commit automático
	 *
	 * @param callback - Função a ser executada na transação
	 * @returns Resultado do callback
	 * @throws Propaga qualquer erro do callback (após rollback)
	 */
	run<T>(callback: () => Promise<T>): Promise<T>;

	/**
	 * Retorna o cliente de transação atual (se houver)
	 * Caso contrário, retorna o cliente normal
	 *
	 * ✅ Usado internamente pelo BaseRepository
	 * ⚠️ Evite usar diretamente - prefira this.db no repository
	 */
	getTransactionClient(): DatabaseTransactionClient | DrizzleClient;

	/**
	 * ✅ Verifica se estamos dentro de uma transação
	 *
	 * Útil para debug e validações
	 */
	isInTransaction(): boolean;
}
\n\n===== FILE: ./src/infrastructure/handlers/errors.handler.ts =====
import type { FastifyError, FastifyReply, FastifyRequest } from "fastify";
import { ZodError } from "zod"; // Ajuste o import se necessário
import { DomainError, TooManyRequestsError } from "../../core/errors/index";
import { env } from "../../shared/config/index";
import { ErrorCodes } from "../../shared/helpers/response.helper";

const isDev = env.NODE_ENV === "development";

export async function errorHandler(
	error: FastifyError,
	request: FastifyRequest,
	reply: FastifyReply,
): Promise<void> {
	// Mudamos para void

	// Log rico: Ajuda MUITO no debug em produção
	request.log.error({
		err: {
			name: error.name,
			message: error.message,
			code: error.code,
			stack: isDev ? error.stack : undefined,
		},
		request: {
			url: request.url,
			method: request.method,
			// Usamos optional chaining para evitar erro se o user não existir
			userId: request.user?.getId?.(),
		},
	});

	// 1. Zod Validation Errors
	if (error instanceof ZodError) {
		return reply.status(400).send({
			success: false,
			code: ErrorCodes.VALIDATION_ERROR,
			error: "Dados de entrada inválidos",
			details: error.issues.map((issue) => ({
				field: issue.path.join("."),
				message: issue.message,
			})),
		});
	}

	// 2. Fastify/AJV Native Validation
	if (error.validation) {
		return reply.status(400).send({
			success: false,
			code: ErrorCodes.VALIDATION_ERROR,
			error: "Erro de schema",
			details: error.validation.map((err) => ({
				field: err.instancePath || "body",
				message: err.message,
			})),
		});
	}

	// 3. Erros de Domínio/Regra de Negócio (Erros esperados)
	if (error instanceof DomainError) {
		return reply.status(error.statusCode || 400).send({
			success: false,
			code: error.code,
			error: error.message,
			details: error.details,
		});
	}

	if (error instanceof TooManyRequestsError) {
		return reply.status(429).send({
			success: false,
			error: {
				message: error.message,
				code: "TOO_MANY_REQUESTS",
			},
		});
	}

	// 4. Erros Desconhecidos (500)
	const statusCode = error.statusCode || 500;
	return reply.status(statusCode).send({
		success: false,
		code: ErrorCodes.INTERNAL_ERROR,
		error: statusCode >= 500 ? "Internal Server Error" : error.message,
		...(isDev && {
			details: {
				message: error.message,
				stack: error.stack,
			},
		}),
	});
}
\n\n===== FILE: ./src/infrastructure/providers/email/base-email.provider.ts =====
export abstract class BaseEmailProvider {
	protected abstract sendRaw(data: {
		to: string;
		subject: string;
		html: string;
	}): Promise<void>;
	public abstract getName(): string;

	async sendVerificationCode(
		to: string,
		code: string,
		expiresAt: Date,
	): Promise<void> {
		const html = `
      <div style="font-family: sans-serif;">
        <h2>Seu código: <span style="color: #4CAF50;">${code}</span></h2>
        <p>Expira às: ${expiresAt.toLocaleTimeString("pt-BR")}</p>
      </div>
    `;
		await this.sendRaw({ to, subject: "Código de Verificação", html });
	}

	async sendPasswordResetCode(
		to: string,
		code: string,
		expiresAt: Date,
	): Promise<void> {
		const html = `
      <div style="font-family: sans-serif;">
        <h2>Seu código de redefinição de senha: <span style="color: #4CAF50;">${code}</span></h2>
        <p>Expira às: ${expiresAt.toLocaleTimeString("pt-BR")}</p>
      </div>
    `;
		await this.sendRaw({ to, subject: "Código de Redefinição de Senha", html });
	}

	async sendAccountCreated(to: string, username: string): Promise<void> {
		const html = `<h1>🎮 Bem-vindo, ${username}!</h1><p>Sua conta está pronta.</p>`;
		await this.sendRaw({ to, subject: "Bem-vindo ao MasterSindico!", html });
	}

	protected parseUserAgent(ua?: string): string {
		if (!ua) return "Dispositivo desconhecido";
		if (ua.includes("Chrome")) return "🌐 Chrome";
		if (ua.includes("Mobile")) return "📱 Mobile";
		return "💻 Navegador";
	}
}
\n\n===== FILE: ./src/infrastructure/providers/email/email.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asClass } from "awilix";
import type { FastifyInstance } from "fastify";
import type { Module } from "../../../shared/module/module.interface.js";
import { NodemailerEmailProvider } from "./nodemailer.email-provider.js";
import { ResendEmailProvider } from "./resend.email-provider.js";
import { ResilientEmailProvider } from "./resilient-email.provider.js";

export class EmailInfraModule implements Module {
	async register(app: FastifyInstance): Promise<void> {
		// Registrar os provedores individuais
		diContainer.register({
			resend: asClass(ResendEmailProvider).singleton(),
			gmail: asClass(NodemailerEmailProvider).singleton(),
		});

		// Registrar o provedor resiliente com injeção explícita
		diContainer.register({
			emailProvider: asClass(ResilientEmailProvider, {
				injector: () => ({
					resend: diContainer.resolve("resend"),
					nodemailer: diContainer.resolve("gmail"),
				}),
			}).singleton(),
		});

		app.log.info("✅ EmailInfraModule registered");
	}

	async bootstrap(_app: FastifyInstance): Promise<void> {}
}
\n\n===== FILE: ./src/infrastructure/providers/email/nodemailer.email-provider.ts =====
import nodemailer from "nodemailer";
import { env } from "../../../shared/config/index.js";
import { BaseEmailProvider } from "./base-email.provider.js";

export class NodemailerEmailProvider extends BaseEmailProvider {
	private transporter: nodemailer.Transporter;

	constructor() {
		super();
		this.transporter = nodemailer.createTransport({
			service: "gmail",
			auth: { user: env.GMAIL_USER, pass: env.GMAIL_APP_PASSWORD },
		});
	}

	getName() {
		return "Gmail (Backup)";
	}

	protected async sendRaw({
		to,
		subject,
		html,
	}: {
		to: string;
		subject: string;
		html: string;
	}): Promise<void> {
		await this.transporter.sendMail({
			from: `"My Sindico" <${env.GMAIL_USER}>`,
			to,
			subject,
			html,
		});
	}
}
\n\n===== FILE: ./src/infrastructure/providers/email/resend.email-provider.ts =====
import { Resend } from "resend";
import { env } from "../../../shared/config/index.js";
import { BaseEmailProvider } from "./base-email.provider.js";

export class ResendEmailProvider extends BaseEmailProvider {
	private client: Resend;

	constructor() {
		super();
		this.client = new Resend(env.RESEND_API_KEY);
	}

	getName() {
		return "Resend (Primary)";
	}

	protected async sendRaw({
		to,
		subject,
		html,
	}: {
		to: string;
		subject: string;
		html: string;
	}): Promise<void> {
		const { error } = await this.client.emails.send({
			from: `MasterSindico <${env.RESEND_SENDER_EMAIL}>`,
			to,
			subject,
			html,
		});
		if (error) throw new Error(`Resend Error: ${error.message}`);
	}
}
\n\n===== FILE: ./src/infrastructure/providers/email/resilient-email.provider.ts =====
import type { IEmailProvider } from "../../../shared/providers/email.provider.js";

interface ResilientEmailProviderDependencies {
	resend: IEmailProvider;
	nodemailer: IEmailProvider;
}

export class ResilientEmailProvider implements IEmailProvider {
	private providers: IEmailProvider[];

	constructor({ resend, nodemailer }: ResilientEmailProviderDependencies) {
		this.providers = [resend, nodemailer];
	}

	getName() {
		return "Resilient System";
	}

	// Encaminha qualquer chamada para o loop de failover
	async sendVerificationCode(to: string, code: string, expiresAt: Date) {
		await this.executeFailover((p) =>
			p.sendVerificationCode(to, code, expiresAt),
		);
	}

	async sendPasswordResetCode(to: string, code: string, expiresAt: Date) {
		await this.executeFailover((p) =>
			p.sendPasswordResetCode(to, code, expiresAt),
		);
	}

	async sendAccountCreatedNotification(to: string, username: string) {
		await this.executeFailover((p) =>
			p.sendAccountCreatedNotification(to, username),
		);
	}

	async sendNewLoginNotification(
		to: string,
		username: string,
		loginDate: Date,
		ipAddress?: string,
		userAgent?: string,
	): Promise<void> {
		await this.executeFailover((p) =>
			p.sendNewLoginNotification(to, username, loginDate, ipAddress, userAgent),
		);
	}

	private async executeFailover(
		fn: (p: IEmailProvider) => Promise<void>,
	): Promise<void> {
		for (const provider of this.providers) {
			try {
				await fn(provider);
				console.log(`✅ Email enviado via ${provider.getName()}`);
				return;
			} catch (err) {
				console.warn(
					`⚠️ Falha no ${provider.getName()}: ${(err as Error).message}`,
				);
			}
		}
		throw new Error("FALHA CRÍTICA: Todos os provedores de e-mail falharam.");
	}
}
\n\n===== FILE: ./src/infrastructure/health/health.route.ts =====
import type { FastifyInstance } from "fastify";
import { z } from "zod";

const healthResponseSchema = z.object({
	status: z.literal("ok"),
	timestamp: z.string(),
	uptime: z.number(),
	environment: z.string(),
});

export async function healthRoute(app: FastifyInstance) {
	app.get(
		"/health",
		{
			schema: {
				tags: ["Health"],
				description: "Health check endpoint",
				response: {
					200: healthResponseSchema,
				},
			},
		},
		async () => {
			return {
				status: "ok" as const,
				timestamp: new Date().toISOString(),
				uptime: process.uptime(),
				environment: process.env.NODE_ENV || "development",
			};
		},
	);
}
\n\n===== FILE: ./src/modules/app.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asValue } from "awilix";
import type { FastifyInstance } from "fastify";
import { DrizzleModule } from "../infrastructure/database/drizzle/drizzle.module";
import { EmailInfraModule } from "../infrastructure/providers/email/email.module";
import type { Module } from "../shared/module/module.interface";
import { AuthModule } from "./auth/auth.module";
import { ProfileModule } from "./profile/profile.module";
import { UserModule } from "./user/user.module";

export class AppModule implements Module {
	private infraModules: Module[] = [];
	private featureModules: Module[] = [];

	constructor() {
		this.infraModules = [new DrizzleModule(), new EmailInfraModule()];

		this.featureModules = [
			new UserModule(),
			new AuthModule(),
			new ProfileModule(),
		];
	}

	async register(app: FastifyInstance): Promise<void> {
		diContainer.register({
			// 1. Logger padrão para serviços que rodam fora do request (como Jobs)
			logger: asValue(app.log),
		});

		// 2. Hook para injetar o logger de contexto (RequestId)
		app.addHook("onRequest", async (request, _reply) => {
			// Isso permite que serviços SCOPED usem o log com ID da requisição
			request.diScope.register({
				logger: asValue(request.log),
			});
		});

		app.log.info("📦 Registrando módulos de infraestrutura...");

		for (const module of this.infraModules) {
			await module.register(app);
			app.log.info(`✅ ${module.constructor.name} registrado`);
		}

		app.log.info("📦 Registrando módulos de features...");

		for (const module of this.featureModules) {
			await module.register(app);
			app.log.info(`✅ ${module.constructor.name} registrado`);
		}
	}

	async bootstrap(app: FastifyInstance): Promise<void> {
		const allModules = [...this.infraModules, ...this.featureModules];

		for (const module of allModules) {
			if (module.bootstrap) {
				await module.bootstrap(app);
			}
		}
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/jobs/session.cleanup.ts =====
import {
	BaseTask,
	type TaskDependencies,
} from "../../../../core/contracts/base-task.js";
import type { ISessionRepository } from "../../domain/repositories/session.repository.js";

interface SessionCleanupDeps extends TaskDependencies {
	sessionRepository: ISessionRepository;
}

export class SessionCleanupTask extends BaseTask {
	private readonly sessionRepository: ISessionRepository;

	constructor(deps: SessionCleanupDeps) {
		super(deps);
		this.sessionRepository = deps.sessionRepository;
	}

	async handleCron(): Promise<void> {
		this.logger.info("SessionCleanupTask: Iniciando limpeza...");

		try {
			await this.unitOfWork.run(async () => {
				await this.sessionRepository.deleteExpired();
			});
			this.logger.info("SessionCleanupTask: Sucesso.");
		} catch (error) {
			this.logger.error({ err: error }, "SessionCleanupTask: Falha.");
		}
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/jobs/verification.cleanup.ts =====
import {
	BaseTask,
	type TaskDependencies,
} from "../../../../core/contracts/base-task";
import type { IVerificationRepository } from "../../domain/repositories/verification.repository";

interface VerificationCleanupDeps extends TaskDependencies {
	verificationRepository: IVerificationRepository;
}

export class VerificationCleanupTask extends BaseTask {
	private readonly verificationRepository: IVerificationRepository;

	constructor(deps: VerificationCleanupDeps) {
		super(deps);
		this.verificationRepository = deps.verificationRepository;
	}

	async handleCron(): Promise<void> {
		this.logger.info("VerificationCleanupTask: Iniciando limpeza...");

		try {
			await this.unitOfWork.run(async () => {
				await this.verificationRepository.deleteExpired();
			});
		} catch (error) {
			this.logger.error({ err: error }, "VerificationCleanupTask: Falha.");
		}
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/database/drizzle/repositories/account.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository.js";
import { NotFoundError } from "../../../../../../core/errors/index.js";
import { accounts } from "../../../../../../infrastructure/database/drizzle/schema/auth/accounts.js";
import type { BaseAccount } from "../../../../domain/entities/base-account.entity.js";
import { LocalAccount } from "../../../../domain/entities/local-account.entity.js";
import { OAuthAccount } from "../../../../domain/entities/oauth-account.entity.js";
import type { IAccountRepository } from "../../../../domain/repositories/account.repository.js";
import { PasswordVO } from "../../../../domain/value-objects/password.vo.js";

export class AccountRepositoryImpl
	extends BaseRepository
	implements IAccountRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<BaseAccount | null> {
		const result = await this.db.query.accounts.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByProviderAccount(
		providerId: string,
		accountId: string,
	): Promise<BaseAccount | null> {
		const result = await this.db.query.accounts.findFirst({
			where: {
				providerId: providerId,
				accountId: accountId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<BaseAccount[]> {
		const results = await this.db.query.accounts.findMany({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return results.map((result) => this.toDomain(result));
	}

	// ==================== COMMANDS ====================
	async save(account: BaseAccount): Promise<void> {
		const data = this.toPersistence(account);

		await this.db
			.insert(accounts)
			.values(data)
			.onConflictDoUpdate({
				target: accounts.id,
				set: {
					// ✅ SÓ campos mutáveis (OAuth tokens podem mudar)
					accessToken: data.accessToken,
					refreshToken: data.refreshToken,
					idToken: data.idToken,
					accessTokenExpiresAt: data.accessTokenExpiresAt,
					refreshTokenExpiresAt: data.refreshTokenExpiresAt,
					scope: data.scope,
					password: data.password,
					updatedAt: new Date(),
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(accounts)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(accounts.id, id));
	}

	async softDeleteByUserId(userId: string): Promise<void> {
		await this.db
			.update(accounts)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(accounts.userId, userId));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(accounts)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(accounts.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(accounts).where(eq(accounts.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof accounts.$inferSelect): BaseAccount {
		// Se tem senha, é LocalAccount
		if (raw.password) {
			return new LocalAccount(
				raw.id,
				raw.userId,
				PasswordVO.fromHash(raw.password),
				raw.createdAt,
				raw.updatedAt,
				raw.deletedAt,
			);
		}

		// Se tem tokens de OAuth, é OAuthAccount
		if (raw.accessToken || raw.refreshToken || raw.idToken) {
			return new OAuthAccount(
				raw.id,
				raw.accountId,
				raw.providerId,
				raw.userId,
				raw.accessToken ?? undefined,
				raw.refreshToken ?? undefined,
				raw.idToken ?? undefined,
				raw.accessTokenExpiresAt ?? undefined,
				raw.refreshTokenExpiresAt ?? undefined,
				raw.scope ?? undefined,
				raw.createdAt,
				raw.updatedAt,
				raw.deletedAt,
			);
		}

		throw new NotFoundError("Account type not recognized");
	}

	private toPersistence(account: BaseAccount): typeof accounts.$inferInsert {
		const baseData = {
			id: account.getId(),
			accountId: account.getAccountId(),
			providerId: account.getProviderId(),
			userId: account.getUserId(),
			createdAt: account.getCreatedAt(),
			updatedAt: account.getUpdatedAt(),
			deletedAt: account.getDeletedAt(),
		};

		// LocalAccount
		if (account instanceof LocalAccount) {
			return {
				...baseData,
				password: account.getPassword().getValue(),
				// Campos OAuth nulos
				accessToken: null,
				refreshToken: null,
				idToken: null,
				accessTokenExpiresAt: null,
				refreshTokenExpiresAt: null,
				scope: null,
			};
		}

		// OAuthAccount
		if (account instanceof OAuthAccount) {
			return {
				...baseData,
				accessToken: account.getAccessToken() ?? null,
				refreshToken: account.getRefreshToken() ?? null,
				idToken: account.getIdToken() ?? null,
				accessTokenExpiresAt: account.getAccessTokenExpiresAt() ?? null,
				refreshTokenExpiresAt: account.getRefreshTokenExpiresAt() ?? null,
				scope: account.getScope() ?? null,
				// Campos de senha nulos
				password: null,
			};
		}

		throw new NotFoundError(
			`Account type not recognized: ${account.constructor.name}`,
		);
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/database/drizzle/repositories/session.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { and, eq, lt, ne } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { sessions } from "../../../../../../infrastructure/database/drizzle/schema";
import { Session } from "../../../../domain/entities/session.entity";
import type { ISessionRepository } from "../../../../domain/repositories/session.repository";

export class SessionRepositoryImpl
	extends BaseRepository
	implements ISessionRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findOneById(id: string): Promise<Session | null> {
		const result = await this.db.query.sessions.findFirst({
			where: { id },
		});

		return result ? this.toDomain(result) : null;
	}

	async findAllByUserId(userId: string): Promise<Session[]> {
		const results = await this.db.query.sessions.findMany({
			where: { userId },
		});

		return results.map((r) => this.toDomain(r));
	}

	// ==================== COMMANDS ====================
	async save(session: Session): Promise<void> {
		await this.db
			.insert(sessions)
			.values(this.toPersistence(session))
			.onConflictDoUpdate({
				target: sessions.id,
				set: this.toPersistence(session),
			});
	}

	async deleteOneById(id: string): Promise<void> {
		await this.db.delete(sessions).where(eq(sessions.id, id));
	}

	async deleteAllByUserId(userId: string): Promise<void> {
		await this.db.delete(sessions).where(eq(sessions.userId, userId));
	}

	async deleteExpired(): Promise<void> {
		await this.db.delete(sessions).where(lt(sessions.expiresAt, new Date()));
	}

	async deleteManyExcept(
		userId: string,
		exceptSessionId: string,
	): Promise<void> {
		await this.db
			.delete(sessions)
			.where(
				and(eq(sessions.userId, userId), ne(sessions.id, exceptSessionId)),
			);
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof sessions.$inferSelect): Session {
		return new Session(
			raw.id,
			raw.userId,
			new Date(raw.expiresAt),
			new Date(raw.createdAt),
			new Date(raw.updatedAt),
			raw.ipAddress ?? undefined,
			raw.userAgent ?? undefined,
		);
	}

	private toPersistence(session: Session): typeof sessions.$inferInsert {
		return {
			id: session.getId(),
			userId: session.getUserId(),
			expiresAt: session.getExpiresAt(),
			createdAt: session.getCreatedAt(),
			updatedAt: session.getUpdatedAt(),
			ipAddress: session.getIpAddress(),
			userAgent: session.getUserAgent(),
		};
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/database/drizzle/repositories/verification.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { and, eq, lt } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { verifications } from "../../../../../../infrastructure/database/drizzle/schema";
import { Verification } from "../../../../domain/entities/verification.entity";
import type { IVerificationRepository } from "../../../../domain/repositories/verification.repository";

export class VerificationRepositoryImpl
	extends BaseRepository
	implements IVerificationRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findByValue(value: string): Promise<Verification | null> {
		const result = await this.db.query.verifications.findFirst({
			where: { value },
		});

		return result ? this.toDomain(result) : null;
	}

	async findLatestByIdentifier(
		identifier: string,
	): Promise<Verification | null> {
		const result = await this.db.query.verifications.findFirst({
			where: { identifier },
			orderBy: {
				createdAt: "desc",
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByValueAndIdentifier(
		value: string,
		identifier: string,
	): Promise<Verification | null> {
		const result = await this.db.query.verifications.findFirst({
			where: {
				value,
				identifier,
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(verification: Verification): Promise<void> {
		await this.db
			.insert(verifications)
			.values(this.toPersistence(verification))
			.onConflictDoUpdate({
				target: verifications.id,
				set: this.toPersistence(verification),
			});
	}

	async deleteByIdentifier(identifier: string): Promise<void> {
		await this.db
			.delete(verifications)
			.where(eq(verifications.identifier, identifier));
	}

	async deleteByValueAndIdentifier(
		value: string,
		identifier: string,
	): Promise<void> {
		await this.db
			.delete(verifications)
			.where(
				and(
					eq(verifications.value, value),
					eq(verifications.identifier, identifier),
				),
			);
	}

	async deleteExpired(): Promise<void> {
		await this.db
			.delete(verifications)
			.where(lt(verifications.expiresAt, new Date()));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof verifications.$inferSelect): Verification {
		return new Verification(
			raw.id,
			raw.identifier,
			raw.value,
			raw.expiresAt,
			raw.createdAt,
		);
	}
	private toPersistence(
		verification: Verification,
	): typeof verifications.$inferInsert {
		return {
			id: verification.getId(),
			identifier: verification.getIdentifier(),
			value: verification.getValue(),
			expiresAt: verification.getExpiresAt(),
			createdAt: verification.getCreatedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/auth/infrastructure/http/routes/auth.routes.ts =====
import type { FastifyInstance } from "fastify";
import type { ZodTypeProvider } from "fastify-type-provider-zod";
import {
	authResponseSchema,
	forgotPasswordResponseSchema,
	forgotPasswordSchema,
	getSessionResponseSchema,
	listSessionsResponseSchema,
	resetPasswordResponseSchema,
	resetPasswordSchema,
	revokeSessionBodySchema,
	SignInSchema,
	sendVerificationResponseSchema,
	sendVerificationSchema,
	signUpResponseSchema,
	signUpSchema,
	verifyCodeResponseSchema,
	verifyCodeSchema,
} from "mastersindico-schemas";
import { z } from "zod";
import { success } from "../../../../../shared/helpers/response.helper";
import { authenticate } from "../../../../../shared/hooks/authenticate.hook";
import {
	apiErrorSchema,
	apiSuccessSchema,
} from "../../../../../shared/types/api-response";

const AUTH_RATE_LIMITS = {
	signIn: { max: 5, timeWindow: "1 minute" },
	signUp: { max: 3, timeWindow: "1 minute" },
	forgotPassword: { max: 3, timeWindow: "5 minutes" },
	verifyEmail: { max: 5, timeWindow: "1 minute" },
	sendVerification: { max: 3, timeWindow: "5 minutes" },
	resetPassword: { max: 3, timeWindow: "5 minutes" },
} as const;

export async function authRoutes(app: FastifyInstance) {
	const typedApp = app.withTypeProvider<ZodTypeProvider>();

	typedApp.post(
		"/sign-up/email",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.signUp,
			},
			schema: {
				tags: ["Auth"],
				body: signUpSchema,
				summary: "Cria uma nova conta de usuário",
				description:
					"Registra um novo usuário no sistema a partir das informações fornecidas. Após a criação da conta, um código de verificação é enviado para o e-mail informado, sendo necessário confirmar o cadastro antes do primeiro login. O processo pode falhar caso os dados sejam inválidos, o e-mail já esteja em uso ou as regras de segurança não sejam atendidas.",
				response: {
					201: apiSuccessSchema(signUpResponseSchema),
					401: apiErrorSchema,
					403: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { signUpUseCase } = request.diScope.cradle;

			const output = await signUpUseCase.execute(request.body);

			return reply.status(201).send(success(output));
		},
	);

	typedApp.post(
		"/sign-in/email",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.signIn,
			},
			schema: {
				tags: ["Auth"],
				body: SignInSchema,
				summary: "Autentica um usuário existente e inicia uma sessão",
				description:
					"Realiza a autenticação do usuário a partir de suas credenciais válidas (e-mail/username e senha). Em caso de sucesso, retorna os tokens de autenticação e os dados básicos da sessão ativa. Falhas de autenticação podem ocorrer por credenciais inválidas, conta inexistente ou conta não verificada.",
				response: {
					201: apiSuccessSchema(authResponseSchema),
					401: apiErrorSchema,
					403: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { signInUseCase, authService } = request.diScope.cradle;

			const metadata = {
				ipAddress: request.ip,
				userAgent: request.headers["user-agent"],
			};

			const output = await signInUseCase.execute(request.body, metadata);

			await authService.handleSessionResponse(reply, output.session);

			return reply.status(201).send(success(output));
		},
	);

	typedApp.post(
		"/sign-out",
		{
			schema: {
				tags: ["Auth"],
				summary: "Encerrar sessão do usuário",
				description:
					"Invalida a sessão atual no banco de dados e remove o cookie de autenticação do navegador do usuário.",
				response: {
					201: apiSuccessSchema(z.any()),
					401: apiErrorSchema,
					403: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { signOutUseCase, authService } = request.diScope.cradle;

			const sessionId = request.session.getId();

			await authService.clearSessionCookie(reply);

			const output = await signOutUseCase.execute(sessionId);

			return reply.status(201).send(success(output));
		},
	);

	typedApp.post(
		"/send-verification-email",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.sendVerification,
			},
			schema: {
				tags: ["Auth"],
				body: sendVerificationSchema,
				summary: "Envia um e-mail de verificação para o usuário",
				description:
					"Gera e envia um novo código de verificação de e-mail associado ao usuário. O código possui tempo de expiração e invalida códigos anteriores ainda não utilizados. A sessão do usuário pode ser atualizada conforme a política de autenticação vigente.",
				response: {
					201: apiSuccessSchema(sendVerificationResponseSchema),
					401: apiErrorSchema,
					403: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { sendVerificationCodeUseCase } = request.diScope.cradle;

			const metadata = {
				ipAddress: request.ip,
				userAgent: request.headers["user-agent"],
			};

			const output = await sendVerificationCodeUseCase.execute(
				request.body,
				metadata,
			);

			return reply.status(201).send(success(output));
		},
	);

	typedApp.post(
		"/verify-email",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.verifyEmail,
			},
			schema: {
				tags: ["Auth"],
				summary: "Confirma o endereço de e-mail do usuário",
				description:
					"Valida o código de verificação enviado para o e-mail do usuário durante o processo de cadastro. Após a confirmação bem-sucedida, a conta é marcada como verificada e uma sessão é criada automaticamente. O processo pode falhar caso o código seja inválido, expirado ou já tenha sido utilizado.",
				body: verifyCodeSchema,
				response: {
					201: apiSuccessSchema(verifyCodeResponseSchema),
					401: apiErrorSchema,
					403: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { verifyCodeUseCase, authService } = request.diScope.cradle;

			const metadata = {
				ipAddress: request.ip,
				userAgent: request.headers["user-agent"],
			};

			const output = await verifyCodeUseCase.execute(request.body, metadata);

			if (output.session) {
				await authService.handleSessionResponse(reply, output.session);
			}

			return reply.status(201).send(success(output));
		},
	);

	typedApp.post(
		"/forgot-password",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.forgotPassword,
			},
			schema: {
				tags: ["Auth"],
				summary: "Solicita recuperação de senha",
				description:
					"Gera um código de 6 dígitos e envia para o e-mail do morador.",
				body: forgotPasswordSchema,
				response: {
					200: apiSuccessSchema(forgotPasswordResponseSchema),
					401: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { forgotPasswordUseCase } = request.diScope.cradle;

			const output = await forgotPasswordUseCase.execute(request.body);

			return reply.send(success(output));
		},
	);

	typedApp.post(
		"/reset-password",
		{
			config: {
				rateLimit: AUTH_RATE_LIMITS.resetPassword,
			},
			schema: {
				tags: ["Auth"],
				summary: "Redefine a senha do usuário",
				description:
					"Valida o token recebido e atualiza a senha na LocalAccount, revogando todas as sessões.",
				body: resetPasswordSchema,
				response: {
					200: apiSuccessSchema(resetPasswordResponseSchema),
					401: apiErrorSchema,
					400: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { resetPasswordUseCase } = request.diScope.cradle;

			const output = await resetPasswordUseCase.execute(request.body);

			return reply.send(success(output));
		},
	);

	typedApp.get(
		"/get-session",
		{
			schema: {
				tags: ["Auth"],
				summary: "Obtém a sessão autenticada atual",
				description:
					"Recupera os dados da sessão associada ao token de autenticação informado, seja via cookie ou header Authorization. Caso o token seja válido e a sessão esteja ativa, retorna as informações do usuário autenticado e o estado da sessão.",
				response: {
					200: apiSuccessSchema(getSessionResponseSchema),
					401: apiErrorSchema,
				},
			},
		},
		async (request, reply) => {
			const { getSessionUseCase } = request.diScope.cradle;

			const token =
				request.cookies.session_token ||
				request.headers.authorization?.replace("Bearer ", "");

			const output = await getSessionUseCase.execute({ token });
			return reply.send(success(output));
		},
	);

	typedApp.post(
		"/sessions/revoke",
		{
			schema: {
				summary: "Revoga uma sessão específica",
				tags: ["Auth"],
				body: revokeSessionBodySchema,
				response: {
					200: apiSuccessSchema(z.any()),
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { revokeSessionUseCase } = request.diScope.cradle;
			const userId = request.user.getId();
			const { sessionId } = request.body;

			const output = await revokeSessionUseCase.execute({
				sessionId,
				userId,
			});

			return reply.send(success(output));
		},
	);

	// Rota 2: Revogar todas as OUTRAS sessões ("Sair de outros dispositivos")
	typedApp.post(
		"/sessions/revoke-others",
		{
			schema: {
				summary: "Revoga todas as sessões exceto a atual",
				tags: ["Auth"],
				response: {
					200: apiSuccessSchema(z.any()),
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { revokeOtherSessionsUseCase } = request.diScope.cradle;
			const userId = request.user.getId();
			const currentSessionId = request.session.getId(); // Vem do AuthMiddleware

			const output = await revokeOtherSessionsUseCase.execute({
				userId,
				sessionId: currentSessionId,
			});

			return reply.send(success(output));
		},
	);

	// Rota 3: Revogar TUDO (Geralmente usado ao trocar senha ou "Sair de tudo")
	typedApp.post(
		"/sessions/revoke-all",
		{
			schema: {
				summary: "Revoga TODAS as sessões do usuário",
				tags: ["Auth"],
				response: {
					205: apiSuccessSchema(
						z.object({
							message: z.string(),
							action: z.literal("REDIRECT_TO_LOGIN"),
						}),
					),
					401: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { revokeAllSessionsUseCase } = request.diScope.cradle;
			const userId = request.user.getId();
			const { authService } = request.diScope.cradle;

			await revokeAllSessionsUseCase.execute({ userId });

			await authService.clearSessionCookie(reply);

			return reply.status(205).send(
				success({
					message: "Todas as sessões foram revogadas com sucesso",
					action: "REDIRECT_TO_LOGIN",
				}),
			);
		},
	);

	typedApp.get(
		"/sessions",
		{
			schema: {
				tags: ["Auth"],
				summary: "Lista todas as sessões ativas do usuário",
				description:
					"Retorna uma lista de todos os dispositivos e navegadores onde o usuário possui uma sessão ativa, indicando qual é a sessão atual.",
				response: {
					200: apiSuccessSchema(listSessionsResponseSchema),
					401: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { listSessionsUseCase } = request.diScope.cradle;

			// Pegamos o ID do usuário e o ID da sessão atual direto do request (injetados pelo hook)
			const userId = request.user.getId();
			const currentSessionId = request.session.getId();

			const output = await listSessionsUseCase.execute(
				{ userId },
				currentSessionId,
			);

			return reply.send(success(output));
		},
	);
}
\n\n===== FILE: ./src/modules/auth/domain/entities/base-account.entity.ts =====
import { BusinessError } from "../../../../core/errors";

export abstract class BaseAccount {
	constructor(
		protected readonly id: string,
		protected readonly accountId: string,
		protected readonly providerId: string,
		protected readonly userId: string,
		protected readonly createdAt: Date = new Date(),
		protected updatedAt: Date = new Date(),
		protected deletedAt: Date | null = null,
	) {}

	getId(): string {
		return this.id;
	}

	getAccountId(): string {
		return this.accountId;
	}

	getProviderId(): string {
		return this.providerId;
	}

	getUserId(): string {
		return this.userId;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	getDeletedAt(): Date | null {
		return this.deletedAt;
	}

	isDeleted(): boolean {
		return this.deletedAt !== null;
	}

	softDelete(): void {
		if (this.deletedAt) {
			throw new BusinessError(
				"Conta já está excluída.",
				"ACCOUNT_ALREADY_DELETED",
			);
		}
		this.deletedAt = new Date();
		this.updatedAt = new Date();
	}

	restore(): void {
		if (!this.deletedAt) {
			throw new BusinessError(
				"Conta não foi excluída.",
				"ACCOUNT_NOT_IS_DELETED",
			);
		}
		this.deletedAt = null;
		this.updatedAt = new Date();
	}

	abstract isPasswordBased(): boolean;
	abstract isOAuthBased(): boolean;

	toDto(): object {
		return {
			id: this.id,
			accountId: this.accountId,
			providerId: this.providerId,
			userId: this.userId,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
			deletedAt: this.deletedAt,
		};
	}
}
\n\n===== FILE: ./src/modules/auth/domain/entities/local-account.entity.ts =====
import { generateId } from "../../../../shared/utils/id-generator";
import type { PasswordVO } from "../value-objects/password.vo";
import { BaseAccount } from "./base-account.entity";

export class LocalAccount extends BaseAccount {
	constructor(
		id: string,
		userId: string,
		private password: PasswordVO,
		createdAt: Date = new Date(),
		updatedAt: Date = new Date(),
		deletedAt: Date | null = null,
	) {
		super(id, userId, "local", userId, createdAt, updatedAt, deletedAt);
	}

	static create(userId: string, password: PasswordVO): LocalAccount {
		return new LocalAccount(generateId(), userId, password);
	}

	getPassword(): PasswordVO {
		return this.password;
	}

	hasPassword(): boolean {
		return true;
	}

	isPasswordBased(): boolean {
		return true;
	}

	isOAuthBased(): boolean {
		return false;
	}

	async verifyPassword(inputPassword: string): Promise<boolean> {
		return this.password.compare(inputPassword);
	}

	changePassword(newPassword: PasswordVO): void {
		this.password = newPassword;
		this.updatedAt = new Date();
	}
}
\n\n===== FILE: ./src/modules/auth/domain/entities/oauth-account.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import { BaseAccount } from "./base-account.entity";

interface OAuthTokens {
	accessToken?: string;
	refreshToken?: string;
	idToken?: string;
	accessTokenExpiresAt?: Date;
	refreshTokenExpiresAt?: Date;
	scope?: string;
}

export class OAuthAccount extends BaseAccount {
	constructor(
		id: string,
		accountId: string,
		providerId: string,
		userId: string,
		private accessToken?: string,
		private refreshToken?: string,
		private idToken?: string,
		private accessTokenExpiresAt?: Date,
		private refreshTokenExpiresAt?: Date,
		private scope?: string,
		createdAt: Date = new Date(),
		updatedAt: Date = new Date(),
		deletedAt: Date | null = null,
	) {
		super(id, accountId, providerId, userId, createdAt, updatedAt, deletedAt);
	}

	static create(
		userId: string,
		providerId: string,
		providerAccountId: string,
		tokens: OAuthTokens,
	): OAuthAccount {
		return new OAuthAccount(
			generateId(),
			providerAccountId,
			providerId,
			userId,
			tokens.accessToken,
			tokens.refreshToken,
			tokens.idToken,
			tokens.accessTokenExpiresAt,
			tokens.refreshTokenExpiresAt,
			tokens.scope,
		);
	}

	isPasswordBased(): boolean {
		return false;
	}

	isOAuthBased(): boolean {
		return true;
	}

	getAccessToken(): string | undefined {
		return this.accessToken;
	}

	getRefreshToken(): string | undefined {
		return this.refreshToken;
	}

	getIdToken(): string | undefined {
		return this.idToken;
	}

	getAccessTokenExpiresAt(): Date | undefined {
		return this.accessTokenExpiresAt;
	}

	getRefreshTokenExpiresAt(): Date | undefined {
		return this.refreshTokenExpiresAt;
	}

	getScope(): string | undefined {
		return this.scope;
	}

	updateTokens(tokens: OAuthTokens): void {
		const hasUpdate = Object.values(tokens).some(
			(value) => value !== undefined,
		);
		if (!hasUpdate) {
			throw new BusinessError("No tokens provided for update");
		}

		if (tokens.accessToken !== undefined) {
			this.accessToken = tokens.accessToken;
		}

		if (tokens.accessTokenExpiresAt !== undefined) {
			this.accessTokenExpiresAt = tokens.accessTokenExpiresAt;
		}

		if (tokens.refreshToken !== undefined) {
			this.refreshToken = tokens.refreshToken;
		}

		if (tokens.refreshTokenExpiresAt !== undefined) {
			this.refreshTokenExpiresAt = tokens.refreshTokenExpiresAt;
		}

		if (tokens.idToken !== undefined) {
			this.idToken = tokens.idToken;
		}

		if (tokens.scope !== undefined) {
			this.scope = tokens.scope;
		}

		this.updatedAt = new Date();
	}

	revokeTokens(): void {
		this.accessToken = undefined;
		this.refreshToken = undefined;
		this.idToken = undefined;
		this.accessTokenExpiresAt = undefined;
		this.refreshTokenExpiresAt = undefined;
		this.updatedAt = new Date();
	}

	isAcesstokenExpired(): boolean {
		if (!this.accessTokenExpiresAt) return false;
		return this.accessTokenExpiresAt < new Date();
	}

	needsTokenRefresh(): boolean {
		if (!this.accessTokenExpiresAt) return false;
		const fiveMinutes = 5 * 60 * 1000;
		return this.accessTokenExpiresAt.getTime() - Date.now() < fiveMinutes;
	}
}
\n\n===== FILE: ./src/modules/auth/domain/entities/session.entity.ts =====
import { BusinessError } from "../../../../core/errors/index.js";

export class Session {
	constructor(
		private readonly id: string,
		private readonly userId: string,
		private expiresAt: Date,
		private readonly createdAt: Date = new Date(),
		private updatedAt: Date = new Date(),
		private ipAddress?: string,
		private userAgent?: string,
	) {}

	// Getters
	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getIpAddress(): string | undefined {
		return this.ipAddress;
	}

	getUserAgent(): string | undefined {
		return this.userAgent;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	getExpiresAt(): Date {
		return this.expiresAt;
	}

	// Comandos
	isExpired(): boolean {
		return this.expiresAt < new Date();
	}

	isValid(): boolean {
		return !this.isExpired();
	}

	invalidate(): void {
		this.expiresAt = new Date();
		this.updatedAt = new Date();
	}

	extend(additionalTime: number): void {
		//additionalTime em milissegundos
		this.expiresAt = new Date(this.expiresAt.getTime() + additionalTime);
		this.updatedAt = new Date();
	}

	extendTo(newExpiresAt: Date): void {
		if (newExpiresAt <= new Date()) {
			throw new BusinessError(
				"A nova data de expiração deve ser uma data futura.",
				"INVALID_EXPIRATION",
			);
		}
		this.expiresAt = newExpiresAt;
		this.updatedAt = new Date();
	}

	toDTO(currentToken?: string) {
		return {
			id: this.id,
			userId: this.userId,
			token: currentToken ?? "",
			expiresAt: this.expiresAt,
			ipAddress: this.ipAddress ?? null,
			userAgent: this.userAgent ?? null,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}
}
\n\n===== FILE: ./src/modules/auth/domain/entities/verification.entity.ts =====
import {
	BusinessError,
	UnauthorizedError,
} from "../../../../core/errors/index.js";

export class Verification {
	constructor(
		private readonly id: string,
		private readonly identifier: string,
		private readonly value: string,
		private expiresAt: Date,
		private readonly createdAt: Date = new Date(),
	) {}

	public getId(): string {
		return this.id;
	}

	public getIdentifier(): string {
		return this.identifier;
	}

	public getValue(): string {
		return this.value;
	}

	public getExpiresAt(): Date {
		return this.expiresAt;
	}

	public getCreatedAt(): Date {
		return this.createdAt;
	}

	//Comandos
	public isExpired(): boolean {
		return this.expiresAt < new Date();
	}

	public isValid(): boolean {
		return !this.isExpired();
	}

	/**
	 * Valida se o código bate e se ainda pode ser usado.
	 * Note que ela não "consome" ainda, apenas verifica.
	 */
	public verify(inputValue: string): void {
		if (this.isExpired()) {
			throw new BusinessError("Este código expirou.", "VERIFICATION_EXPIRED");
		}

		if (this.value !== inputValue) {
			throw new UnauthorizedError("Código de verificação incorreto.");
		}
	}
}
\n\n===== FILE: ./src/modules/auth/domain/value-objects/password.vo.ts =====
import * as argon from "argon2";

export class PasswordVO {
	private constructor(private readonly hashedValue: string) {}

	static async create(plain: string): Promise<PasswordVO> {
		if (!plain || plain.length < 8) {
			throw new Error("Password must be at least 8 characters long");
		}
		const hash = await argon.hash(plain);
		return new PasswordVO(hash);
	}

	static fromHash(hash: string): PasswordVO {
		if (!hash) throw new Error("Invalid hash");
		return new PasswordVO(hash);
	}

	getValue(): string {
		return this.hashedValue;
	}

	async compare(plain: string): Promise<boolean> {
		return argon.verify(this.hashedValue, plain);
	}
}
\n\n===== FILE: ./src/modules/auth/domain/repositories/account.repository.ts =====
import type { BaseAccount } from "../entities/base-account.entity";

export interface IAccountRepository {
	findById(id: string): Promise<BaseAccount | null>;
	findByProviderAccount(
		providerId: string,
		accountId: string,
	): Promise<BaseAccount | null>;
	findByUserId(userId: string): Promise<BaseAccount[]>;

	save(account: BaseAccount): Promise<void>;
	softDelete(id: string): Promise<void>;
	softDeleteByUserId(userId: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/auth/domain/repositories/session.repository.ts =====
import type { Session } from "../entities/session.entity.js";

export interface ISessionRepository {
	findOneById(id: string): Promise<Session | null>;
	findAllByUserId(userId: string): Promise<Session[]>;

	save(session: Session): Promise<void>;
	deleteOneById(id: string): Promise<void>;
	deleteAllByUserId(userId: string): Promise<void>;
	deleteExpired(): Promise<void>;
	deleteManyExcept(userId: string, exceptSessionId: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/auth/domain/repositories/verification.repository.ts =====
import type { Verification } from "../entities/verification.entity.js";

export interface IVerificationRepository {
	findByValue(value: string): Promise<Verification | null>;
	findLatestByIdentifier(identifier: string): Promise<Verification | null>;
	findByValueAndIdentifier(
		value: string,
		identifier: string,
	): Promise<Verification | null>;

	save(verification: Verification): Promise<void>;
	deleteByValueAndIdentifier(value: string, identifier: string): Promise<void>;
	deleteByIdentifier(identifier: string): Promise<void>;
	deleteExpired(): Promise<void>;
}
\n\n===== FILE: ./src/modules/auth/domain/services/jwt.service.ts =====
import type { FastifyRequest } from "fastify";
import * as jose from "jose";
import { UnauthorizedError } from "../../../../core/errors/index.js";
import { env } from "../../../../shared/config/index.js";

export interface JwtPayload extends jose.JWTPayload {
	sub: string; // userId
	sid: string; // sessionId
	iat?: number; // issued at
	exp?: number; // expires at
}

export interface TokenPair {
	accessToken: string;
	expiresAt: Date;
}

export class JwtTokenService {
	private readonly secret: Uint8Array;
	constructor() {
		this.secret = new TextEncoder().encode(env.JWT_SECRET);
	}

	async generateToken(
		userId: string,
		sessionId: string,
		expiresAt: Date,
	): Promise<TokenPair> {
		const accessToken = await new jose.SignJWT({ sid: sessionId })
			.setProtectedHeader({ alg: "HS256" })
			.setSubject(userId)
			.setIssuedAt()
			.setExpirationTime(Math.floor(expiresAt.getTime() / 1000))
			.sign(this.secret);

		return {
			accessToken,
			expiresAt,
		};
	}

	/**
	 * Verifica a assinatura E a expiração
	 */
	async verifyToken(token: string): Promise<JwtPayload> {
		try {
			const { payload } = await jose.jwtVerify(token, this.secret, {
				algorithms: ["HS256"],
			});

			return payload as JwtPayload;
		} catch (_err) {
			throw new UnauthorizedError();
		}
	}

	/**
	 * Verifica a assinatura, mas ignora a expiração (Para Refresh Token)
	 */
	async verifyIgnoreExpiration(token: string): Promise<JwtPayload> {
		try {
			const { payload } = await jose.jwtVerify(token, this.secret, {
				algorithms: ["HS256"],
				currentDate: new Date(0), // valida a assinatura mas ignorar o tempo.
			});

			return payload as JwtPayload;
		} catch (_err) {
			throw new UnauthorizedError();
		}
	}

	decode(token: string): JwtPayload | null {
		return jose.decodeJwt(token) as JwtPayload;
	}

	extractToken(request: FastifyRequest): string | undefined {
		// Tenta extrair do Cookie (Web)
		const cookieToken = request.cookies.session_token;
		if (cookieToken) return cookieToken;

		// Tenta extrair do Header (Mobile)
		const authHeader = request.headers.authorization;
		if (authHeader?.startsWith("Bearer ")) {
			return authHeader.substring(7);
		}

		return undefined;
	}
}
\n\n===== FILE: ./src/modules/auth/domain/services/auth.service.ts =====
import type { FastifyReply } from "fastify";
import type { UseCaseDependencies } from "../../../../core/contracts/base-use-case.js";
import { env } from "../../../../shared/config/index.js";
import type { AuthMetadata } from "../../../../shared/contracts/auth-metadata.js";
import type { IEmailProvider } from "../../../../shared/providers/email.provider.js";
import { generateId } from "../../../../shared/utils/id-generator.js";
import type { User } from "../../../user/domain/entities/user.entity.js";
import { Session } from "../entities/session.entity.js";
import { getSessionExpirationDate } from "../helpers/session.helper.js";
import type { ISessionRepository } from "../repositories/session.repository.js";
import type { JwtTokenService } from "./jwt.service.js";

export interface AuthenticatedSession {
	session: Session;
	accessToken: string;
	expiresAt: Date;
}

interface AuthServiceDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
	jwtService: JwtTokenService;
	emailProvider: IEmailProvider;
}

export class AuthService {
	constructor(private readonly deps: AuthServiceDependencies) {}

	createSession(user: User, metadata: AuthMetadata): Session {
		return new Session(
			generateId(),
			user.getId(),
			getSessionExpirationDate(),
			new Date(),
			new Date(),
			metadata.ipAddress,
			metadata.userAgent,
		);
	}

	async createAuthenticatedSession(
		user: User,
		metadata: AuthMetadata,
	): Promise<AuthenticatedSession> {
		// 1. Busca as sessões atuais do usuário
		const activeSessions = await this.deps.sessionRepository.findAllByUserId(
			user.getId(),
		);

		// 2. Se estourou o limite (ex: 5)
		if (activeSessions.length >= env.SESSION_MAX_ACTIVE_SESSIONS) {
			// Ordena pela mais antiga e pega a primeira (ou as primeiras se o estouro for grande)
			const sessionsToDelete = activeSessions
				.sort((a, b) => a.getCreatedAt().getTime() - b.getCreatedAt().getTime())
				.slice(0, activeSessions.length - env.SESSION_MAX_ACTIVE_SESSIONS + 1);

			// Deleta as antigas para dar lugar à nova
			for (const oldSession of sessionsToDelete) {
				await this.deps.sessionRepository.deleteOneById(oldSession.getId());
				this.deps.logger.info(
					{ sessionId: oldSession.getId() },
					"Sessão antiga removida por excesso de dispositivos",
				);
			}
		}

		const session = this.createSession(user, metadata);
		const { accessToken, expiresAt } = await this.deps.jwtService.generateToken(
			user.getId(),
			session.getId(),
			session.getExpiresAt(),
		);

		return { session, accessToken, expiresAt };
	}

	async handleSessionResponse(
		reply: FastifyReply,
		sessionData: { token: string; expiresAt: Date },
	): Promise<void> {
		const request = reply.request;

		// Verifica se é mobile (ajuste a lógica conforme seu app)
		const isMobile =
			request.headers["x-client-type"] === "mobile" ||
			request.headers["user-agent"]?.includes("PostmanRuntime");

		if (!isMobile) {
			reply.setCookie("session_token", sessionData.token, {
				path: "/",
				httpOnly: true,
				secure: process.env.NODE_ENV === "production",
				expires: sessionData.expiresAt,
				sameSite: "lax",
			});
		}
	}

	async notifyAuthentication(
		type: "signup" | "login" | "oauth",
		user: User,
		session: Session,
	): Promise<void> {
		const userEmail = user.getEmail().getValue();

		try {
			this.deps.logger.info(
				{
					userId: user.getId(),
					type,
				},
				`Iniciando notificação de ${type} para o usuário`,
			);

			if (type === "signup" || type === "oauth") {
				await this.deps.emailProvider.sendAccountCreatedNotification(
					userEmail,
					user.getUsername().getValue(),
				);
			} else {
				await this.deps.emailProvider.sendNewLoginNotification(
					userEmail,
					user.getUsername().getValue(),
					session.getCreatedAt(),
					session.getIpAddress(),
					session.getUserAgent(),
				);
			}

			this.deps.logger.info(
				{ userEmail },
				`Notificação de ${type} enviada com sucesso`,
			);
		} catch (error) {
			this.deps.logger.error(
				{
					err: error,
					userId: user.getId(),
					email: userEmail,
					type,
				},
				`FALHA_NOTIFICACAO: Não foi possível enviar e-mail de ${type}`,
			);
		}
	}

	async clearSessionCookie(reply: FastifyReply): Promise<void> {
		reply.clearCookie("session_token", {
			path: "/",
			httpOnly: true,
			secure: process.env.NODE_ENV === "production",
			sameSite: "lax",
		});
	}
}
\n\n===== FILE: ./src/modules/auth/domain/services/username-generator.service.ts =====
import type { FastifyBaseLogger } from "fastify";
import { ConflictError } from "../../../../core/errors/index.js";
import { generatePrefixedSlug } from "../../../../shared/utils/id-generator.js";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository.js";

interface UsernameGeneratorDependencies {
	logger: FastifyBaseLogger;
	userRepository: IUserRepository;
}

export class UsernameGeneratorService {
	private readonly logger: FastifyBaseLogger;
	private readonly userRepository: IUserRepository;

	// ✅ CORREÇÃO: Recebe objeto desestruturado
	constructor({ logger, userRepository }: UsernameGeneratorDependencies) {
		this.logger = logger;
		this.userRepository = userRepository;
	}

	async execute(prefix: "p" | "c" = "p", maxRetries = 5): Promise<string> {
		for (let i = 0; i < maxRetries; i++) {
			const slug = generatePrefixedSlug(prefix);

			// O repositório precisa ter esse método findByUsername
			const existing = await this.userRepository.findByUsername(slug);

			if (!existing) {
				return slug;
			}

			this.logger.warn(`[Username Collision] Tentativa ${i + 1}: ${slug}`);
		}

		throw new ConflictError(
			"Não foi possível gerar um username único. O sistema pode estar com alta taxa de colisão.",
		);
	}
}
\n\n===== FILE: ./src/modules/auth/domain/helpers/session.helper.ts =====
import { env } from "../../../../shared/config/index.js";

const EXPIRATION_MS = env.SESSION_EXPIRATION_DAYS * 24 * 60 * 60 * 1000;

export function getSessionExpirationMs(): number {
	return EXPIRATION_MS;
}

export function getSessionExpirationDate(): Date {
	return new Date(Date.now() + EXPIRATION_MS);
}
\n\n===== FILE: ./src/modules/auth/domain/helpers/verification.helper.ts =====
import { env } from "../../../../shared/config/index.js";

const EXPIRATION_MS = env.VERIFICATION_EXPIRATION_MINS * 60 * 1000;

export function getVerificationExpirationMs(): number {
	return EXPIRATION_MS;
}

export function getVerificationExpirationDate(): Date {
	return new Date(Date.now() + EXPIRATION_MS);
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/sign-up.use-case.ts =====
import type { SignUpDto, SignUpResponseDto } from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import { User } from "../../../user/domain/entities/user.entity";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { EmailVO } from "../../../user/domain/value-objects/email.vo";
import { UsernameVO } from "../../../user/domain/value-objects/username.vo";
import { LocalAccount } from "../../domain/entities/local-account.entity";
import type { IAccountRepository } from "../../domain/repositories/account.repository";
import type { UsernameGeneratorService } from "../../domain/services/username-generator.service";
import { PasswordVO } from "../../domain/value-objects/password.vo";
import type { SendVerificationCodeUseCase } from "./verification/send-verification-code.use-case";

interface SignUpUseCaseDependencies extends UseCaseDependencies {
	usernameGenerator: UsernameGeneratorService;
	accountRepository: IAccountRepository;
	userRepository: IUserRepository;
	sendVerificationCodeUseCase: SendVerificationCodeUseCase;
}

export class SignUpUseCase extends TransactionalUseCase<
	SignUpDto,
	SignUpResponseDto
> {
	constructor(private readonly deps: SignUpUseCaseDependencies) {
		super(deps);
	}

	async execute(data: SignUpDto): Promise<SignUpResponseDto> {
		if (((data as any).role as string) === "admin") {
			throw new BusinessError(
				"Não é permitido criar uma conta de administrador por esta via.",
				"FORBIDDEN_ROLE_REGISTRATION",
			);
		}

		const user = await this.unitOfWork.run(async () => {
			await this.validateUserDoesNotExists(data.email);

			const user = await this.createUser(data);
			const account = await this.createLocalAccount(user, data.password);

			await this.persistUserAndAccount(user, account);

			return user;
		});

		await this.sendVerificationEmail(user);

		return {
			message: "Conta criada com sucesso. Verifique seu e-mail para continuar.",
			email: user.getEmail().getValue(),
			requiresVerification: true,
		};
	}

	private async validateUserDoesNotExists(email: string): Promise<void> {
		const exists = await this.deps.userRepository.findByEmail(email);

		if (exists) {
			throw new BusinessError(
				"Este email já está em uso",
				"EMAIL_ALREADY_EXISTS",
			);
		}
	}

	private async createUser(data: SignUpDto): Promise<User> {
		const rawUsername = await this.deps.usernameGenerator.execute("p");
		return new User(
			generateId(),
			UsernameVO.create(rawUsername),
			data.name,
			EmailVO.create(data.email),
			data.role,
			false,
			null,
			null,
			false,
			null,
			null,
		);
	}

	private async createLocalAccount(
		user: User,
		plainPassword: string,
	): Promise<LocalAccount> {
		const password = await PasswordVO.create(plainPassword);
		return LocalAccount.create(user.getId(), password);
	}

	private async persistUserAndAccount(
		user: User,
		account: LocalAccount,
	): Promise<void> {
		await this.deps.userRepository.save(user);
		await this.deps.accountRepository.save(account);
	}

	private async sendVerificationEmail(user: User): Promise<void> {
		try {
			await this.deps.sendVerificationCodeUseCase.execute({
				email: user.getEmail().getValue(),
			});
			this.logger.info(
				{ email: user.getEmail().getValue() },
				"Verification email sent",
			);
		} catch (err) {
			this.logger.error(
				{ err, email: user.getEmail().getValue() },
				"Failed to send verification email",
			);
		}
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/verification/send-verification-code.use-case.ts =====
import type {
	SendVerificationDto,
	SendVerificationResponseDto,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../../core/contracts/base-use-case.js";
import {
	BusinessError,
	UnauthorizedError,
} from "../../../../../core/errors/index.js";
import type { AuthMetadata } from "../../../../../shared/contracts/auth-metadata.js";
import type { IEmailProvider } from "../../../../../shared/providers/email.provider.js";
import { generateId } from "../../../../../shared/utils/id-generator.js";
import type { IUserRepository } from "../../../../user/domain/repositories/user.repository.js";
import { Verification } from "../../../domain/entities/verification.entity.js";
import { getSessionExpirationDate } from "../../../domain/helpers/session.helper.js";
import type { IVerificationRepository } from "../../../domain/repositories/verification.repository.js";

interface SendVerificationCodeDependencies extends UseCaseDependencies {
	userRepository: IUserRepository;
	verificationRepository: IVerificationRepository;
	emailProvider: IEmailProvider;
}

export class SendVerificationCodeUseCase extends TransactionalUseCase<
	SendVerificationDto,
	SendVerificationResponseDto,
	AuthMetadata
> {
	constructor(private readonly deps: SendVerificationCodeDependencies) {
		super(deps);
	}

	async execute(
		input: SendVerificationDto,
	): Promise<SendVerificationResponseDto> {
		const { email } = input;

		// 1. Lógica de banco dentro da transação
		const { code, expiresAt } = await this.unitOfWork.run(async () => {
			const user = await this.deps.userRepository.findByEmail(email);

			if (!user) {
				// Segurança: logamos o erro mas não damos pista pro atacante
				this.logger.warn(
					{ email },
					"Tentativa de verificação para e-mail inexistente",
				);
				throw new UnauthorizedError("Usuário não encontrado");
			}

			if (user.isEmailVerified()) {
				throw new BusinessError(
					"Este email já está verificado",
					"EMAIL_ALREADY_VERIFIED",
				);
			}

			// Limpa códigos antigos
			await this.deps.verificationRepository.deleteByIdentifier(email);

			// Código de 6 dígitos e expiração curta (Ex: 15 min)
			const code = this.generateCode();
			const expiresAt = getSessionExpirationDate();

			const newVerification = new Verification(
				generateId(),
				email,
				code,
				expiresAt,
			);

			await this.deps.verificationRepository.save(newVerification);
			return { code, expiresAt };
		});

		// 2. Envio de E-mail FORA da transação
		try {
			await this.deps.emailProvider.sendVerificationCode(
				email,
				code,
				expiresAt,
			);
			this.logger.info({ email }, "Código de verificação enviado");
		} catch (error) {
			// Se o e-mail falhar, logamos mas o código já está no banco se o user quiser reenviar
			this.logger.error(
				{ err: error, email },
				"Falha ao enviar e-mail de verificação",
			);
		}

		return {
			message: "Código de verificação enviado com sucesso",
			expiresAt,
		};
	}

	private generateCode(): string {
		return Math.floor(100000 + Math.random() * 900000).toString();
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/verification/verify-code.use-case.ts =====
import type {
	VerifyCodeDto,
	VerifyCodeResponseDto,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../../core/contracts/base-use-case.js";
import { UnauthorizedError } from "../../../../../core/errors/index.js";
import type { AuthMetadata } from "../../../../../shared/contracts/auth-metadata.js";
import type { IUserRepository } from "../../../../user/domain/repositories/user.repository.js";
import type { ISessionRepository } from "../../../domain/repositories/session.repository.js";
import type { IVerificationRepository } from "../../../domain/repositories/verification.repository.js";
import type { AuthService } from "../../../domain/services/auth.service.js";

interface VerifyCodeDependencies extends UseCaseDependencies {
	userRepository: IUserRepository;
	verificationRepository: IVerificationRepository;
	sessionRepository: ISessionRepository;
	authService: AuthService;
}

export class VerifyCodeUseCase extends TransactionalUseCase<
	VerifyCodeDto,
	VerifyCodeResponseDto,
	AuthMetadata
> {
	constructor(protected readonly deps: VerifyCodeDependencies) {
		super(deps);
	}

	async execute(
		data: VerifyCodeDto,
		metadata?: AuthMetadata,
	): Promise<VerifyCodeResponseDto> {
		return await this.unitOfWork.run(async () => {
			const { email, code } = data;

			this.logger.info({ email }, "Iniciando verificação de código");

			// 1. Busca o usuário primeiro
			const user = await this.deps.userRepository.findByEmail(email);
			if (!user) {
				this.logger.warn(
					{ email },
					"Tentativa de verificação para usuário inexistente",
				);
				throw new UnauthorizedError("Código inválido ou expirado");
			}

			// 2. Se já estiver verificado, não cria nova sessão (Idempotência)
			if (user.isEmailVerified()) {
				return {
					message: "Email já verificado anteriormente",
					emailVerified: true,
				};
			}

			// 3. Busca a verificação
			const verification =
				await this.deps.verificationRepository.findLatestByIdentifier(email);

			if (!verification) {
				this.logger.warn({ email }, "Nenhum código encontrado");
				throw new UnauthorizedError("Código inválido ou expirado");
			}

			// 4. Lógica de Domínio (Verifica expiração e valor do código)
			verification.verify(code);

			// 5. Atualiza o Usuário
			user.verifyEmail();
			await this.deps.userRepository.save(user);

			// 6. Limpa a casa (Remove o código usado)
			await this.deps.verificationRepository.deleteByIdentifier(email);

			// 7. Cria sessão autenticada (NOVO - sessão só é criada após verificação)
			if (metadata) {
				const { session, accessToken } =
					await this.deps.authService.createAuthenticatedSession(user, metadata);
				await this.deps.sessionRepository.save(session);

				this.logger.info(
					{ userId: user.getId() },
					"E-mail verificado e sessão criada com sucesso",
				);

				return {
					message: "E-mail verificado com sucesso",
					emailVerified: true,
					session: { token: accessToken },
					user: user.toDTO(),
				};
			}

			this.logger.info(
				{ userId: user.getId() },
				"E-mail verificado com sucesso (sem sessão)",
			);

			return {
				message: "E-mail verificado com sucesso",
				emailVerified: true,
			};
		});
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/revoke-session.use-case.ts =====
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { NotFoundError, UnauthorizedError } from "../../../../core/errors";
import type { ISessionRepository } from "../../domain/repositories/session.repository";

interface RevokeSessionDto {
	sessionId: string;
	userId: string;
}

interface RevokeSessionUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
}

export class RevokeSessionUseCase extends TransactionalUseCase<
	RevokeSessionDto,
	void
> {
	constructor(private readonly deps: RevokeSessionUseCaseDependencies) {
		super(deps);
	}

	async execute(data: RevokeSessionDto): Promise<void> {
		return await this.unitOfWork.run(async () => {
			// 1. Busca a sessão que quer revogar
			const session = await this.deps.sessionRepository.findOneById(
				data.sessionId,
			);

			if (!session) {
				throw new NotFoundError("Sessão não encontrada");
			}

			// 2. Verifica se a sessão pertence ao usuário logado
			if (session.getUserId() !== data.userId) {
				throw new UnauthorizedError(
					"Você não pode revogar sessões de outros usuários",
				);
			}

			// 3. Deleta a sessão
			await this.deps.sessionRepository.deleteOneById(data.sessionId);

			this.logger.info(
				{ sessionId: data.sessionId, userId: data.userId },
				"Sessão revogada com sucesso",
			);
		});
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/revoke-other-sessions.use-case.ts =====
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import type { ISessionRepository } from "../../domain/repositories/session.repository";

interface RevokeOtherSessionsDto {
	userId: string;
	sessionId: string;
}

interface RevokeOtherSessionsUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
}

export class RevokeOtherSessionsUseCase extends TransactionalUseCase<
	RevokeOtherSessionsDto,
	void
> {
	constructor(private readonly deps: RevokeOtherSessionsUseCaseDependencies) {
		super(deps);
	}

	async execute(data: RevokeOtherSessionsDto): Promise<void> {
		return await this.unitOfWork.run(async () => {
			await this.deps.sessionRepository.deleteManyExcept(
				data.userId,
				data.sessionId,
			);

			this.logger.warn(
				{ userId: data.userId, keptSession: data.sessionId },
				"Todas as outras sessões foram revogadas",
			);
		});
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/revoke-all-sessions.use-case.ts =====
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import type { ISessionRepository } from "../../domain/repositories/session.repository";

interface RevokeAllSessionsDto {
	userId: string;
}

interface RevokeAllSessionsUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
}

export class RevokeAllSessionsUseCase extends TransactionalUseCase<
	RevokeAllSessionsDto,
	void
> {
	constructor(private readonly deps: RevokeAllSessionsUseCaseDependencies) {
		super(deps);
	}

	async execute(data: RevokeAllSessionsDto): Promise<void> {
		return await this.unitOfWork.run(async () => {
			await this.deps.sessionRepository.deleteAllByUserId(data.userId);

			this.logger.warn(
				{ userId: data.userId },
				"Todas as sessões do usuário foram revogadas",
			);
		});
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/list-sessions.use-case.ts =====
import type {
	ListSessionsDto,
	ListSessionsResponseDto,
} from "mastersindico-schemas";
import { UAParser } from "ua-parser-js";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import type { ISessionRepository } from "../../domain/repositories/session.repository";

interface ListSessionsUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
}

export class ListSessionsUseCase extends TransactionalUseCase<
	ListSessionsDto,
	ListSessionsResponseDto
> {
	constructor(private readonly deps: ListSessionsUseCaseDependencies) {
		super(deps);
	}

	async execute(
		data: ListSessionsDto,
		currentSessionId?: string,
	): Promise<ListSessionsResponseDto> {
		const sessions = await this.deps.sessionRepository.findAllByUserId(
			data.userId,
		);

		// 1. Mapeia as sessões para o formato do DTO
		const sessionList = sessions
			.filter((session) => !session.isExpired())
			.map((session) => {
				const uaRaw = session.getUserAgent();
				const ua = new UAParser(uaRaw || "").getResult();

				return {
					id: session.getId(),
					token: "********",
					userId: session.getUserId(),
					// ✅ O PULO DO GATO: O Zod quer null, o banco/entidade dá undefined.
					// O '?? null' resolve o erro 2322.
					ipAddress: session.getIpAddress() ?? null,
					userAgent: uaRaw ?? null,
					createdAt: session.getCreatedAt(),
					updatedAt: session.getUpdatedAt(),
					expiresAt: session.getExpiresAt(),
					isCurrent: session.getId() === currentSessionId,
					deviceInfo: {
						browser:
							`${ua.browser.name || "Unknown"} ${ua.browser.version || ""}`.trim(),
						os: `${ua.os.name || "Unknown"} ${ua.os.version || ""}`.trim(),
						device: ua.device.model || "Desktop/Generic",
					},
				};
			});

		// 2. Ordena: A atual sempre no topo, depois as mais recentes
		sessionList.sort((a, b) => {
			if (a.isCurrent) return -1;
			if (b.isCurrent) return 1;
			return b.createdAt.getTime() - a.createdAt.getTime();
		});

		this.logger.info(
			{ userId: data.userId, count: sessionList.length },
			"Sessões listadas com sucesso",
		);

		// ✅ RETORNO CORRETO: Precisa bater com o ListSessionsResponseDto
		return {
			sessions: sessionList,
			total: sessionList.length,
		};
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/forgot-password.use-case.ts =====
import type {
	ForgotPasswordDto,
	ForgotPasswordResponseDto,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { UnauthorizedError } from "../../../../core/errors";
import type { AuthMetadata } from "../../../../shared/contracts/auth-metadata";
import type { IEmailProvider } from "../../../../shared/providers/email.provider";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { Verification } from "../../domain/entities/verification.entity";
import type { IVerificationRepository } from "../../domain/repositories/verification.repository";

interface ForgotPasswordUseCaseDependencies extends UseCaseDependencies {
	userRepository: IUserRepository;
	verificationRepository: IVerificationRepository;
	emailProvider: IEmailProvider;
}

export class ForgotPasswordUseCase extends TransactionalUseCase<
	ForgotPasswordDto,
	ForgotPasswordResponseDto,
	AuthMetadata
> {
	constructor(private readonly deps: ForgotPasswordUseCaseDependencies) {
		super(deps);
	}

	override async execute(
		input: ForgotPasswordDto,
	): Promise<ForgotPasswordResponseDto> {
		const { email } = input;

		const { code, expiresAt } = await this.unitOfWork.run(async () => {
			const user = await this.deps.userRepository.findByEmail(email);

			// 1. Segurança: Se não existir, não avisa (evita User Enumeration)
			if (!user) {
				this.logger.warn(
					{ email },
					"E-mail não cadastrado, mas simulando sucesso.",
				);
				throw new UnauthorizedError("Usuário não encontrado");
			}

			// 2. Limpa resets pendentes
			await this.deps.verificationRepository.deleteByIdentifier(email);

			// Código de 6 dígitos e expiração curta (Ex: 15 min)
			const code = this.generateCode();
			const expiresAt = new Date(Date.now() + 15 * 60 * 1000);

			const newVerification = new Verification(
				generateId(),
				email,
				code,
				expiresAt,
			);
			await this.deps.verificationRepository.save(newVerification);
			return { code, expiresAt };
		});

		try {
			await this.deps.emailProvider.sendPasswordResetCode(
				email,
				code,
				expiresAt,
			);
			this.logger.info({ email }, "Código de redefinição enviado");
		} catch (error) {
			// Se o e-mail falhar, logamos mas o código já está no banco se o user quiser reenviar
			this.logger.error(
				{ err: error, email },
				"Falha ao enviar e-mail de verificação",
			);
		}

		return {
			message: "Código de verificação enviado com sucesso",
			expiresAt,
		};
	}

	private generateCode(): string {
		return Math.floor(100000 + Math.random() * 900000).toString();
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/reset-password.use-case.ts =====
import type {
	ResetPasswordDto,
	ResetPasswordResponseDto,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, UnauthorizedError } from "../../../../core/errors";
import type { AuthMetadata } from "../../../../shared/contracts/auth-metadata";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { LocalAccount } from "../../domain/entities/local-account.entity";
import type { IAccountRepository } from "../../domain/repositories/account.repository";
import type { ISessionRepository } from "../../domain/repositories/session.repository";
import type { IVerificationRepository } from "../../domain/repositories/verification.repository";
import { PasswordVO } from "../../domain/value-objects/password.vo";

interface ResetPasswordUseCaseDependencies extends UseCaseDependencies {
	userRepository: IUserRepository;
	accountRepository: IAccountRepository;
	sessionRepository: ISessionRepository;
	verificationRepository: IVerificationRepository;
}

export class ResetPasswordUseCase extends TransactionalUseCase<
	ResetPasswordDto,
	ResetPasswordResponseDto,
	AuthMetadata
> {
	constructor(protected readonly deps: ResetPasswordUseCaseDependencies) {
		super(deps);
	}

	async execute(input: ResetPasswordDto): Promise<ResetPasswordResponseDto> {
		return await this.unitOfWork.run(async () => {
			const { code, password } = input;

			this.logger.info({ code }, "Iniciando recuperação de senha");

			const verification =
				await this.deps.verificationRepository.findByValue(code);

			if (!verification) {
				this.logger.warn({ code }, "Nenhum código encontrado");
				throw new UnauthorizedError("Código inválido ou expirado");
			}

			const email = verification.getIdentifier();

			// 1. Busca o usuário primeiro
			const user = await this.deps.userRepository.findByEmail(email);
			if (!user) {
				this.logger.warn(
					{ email },
					"Tentativa de recuperação de senha para usuário inexistente",
				);
				throw new UnauthorizedError("Código inválido ou expirado");
			}

			const userAccounts = await this.deps.accountRepository.findByUserId(
				user.getId(),
			);
			const localAccount = userAccounts.find(
				(acc) => acc instanceof LocalAccount,
			) as LocalAccount;

			if (!localAccount) {
				// Se o cara só loga com Google/GitHub, ele não tem senha pra resetar
				throw new BusinessError(
					"Esta conta não possui uma senha definida. Tente entrar via rede social.",
					"NO_LOCAL_ACCOUNT_FOUND",
				);
			}

			const newPasswordVO = await PasswordVO.create(password);
			localAccount.changePassword(newPasswordVO);

			await this.deps.accountRepository.save(localAccount);

			await this.deps.verificationRepository.deleteByIdentifier(email);

			await this.deps.sessionRepository.deleteAllByUserId(user.getId());

			this.logger.info(
				{ userId: user.getId() },
				"Senha da conta local resetada com sucesso",
			);

			return {
				message: "Senha alterada com sucesso",
			};
		});
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/sign-out.use-case.ts =====
import type { SignOutDto, SignOutResponseDto } from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import type { ISessionRepository } from "../../domain/repositories/session.repository";

interface SignOutUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
}

export class SignOutUseCase extends TransactionalUseCase<
	SignOutDto,
	SignOutResponseDto
> {
	constructor(private readonly deps: SignOutUseCaseDependencies) {
		super(deps);
	}

	async execute(input: SignOutDto): Promise<SignOutResponseDto> {
		const { sessionId } = input;
		this.logger.info({ sessionId }, "Iniciando logout da sessão");

		await this.deps.sessionRepository.deleteOneById(sessionId);

		this.logger.info("Sessão encerrada com sucesso");

		return { message: "sucess" };
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/sign-in.use-case.ts =====
import type { AuthResponseDto, SignInDto } from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { UnauthorizedError } from "../../../../core/errors";
import type { AuthMetadata } from "../../../../shared/contracts/auth-metadata";
import type { User } from "../../../user/domain/entities/user.entity";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { LocalAccount } from "../../domain/entities/local-account.entity";
import type { IAccountRepository } from "../../domain/repositories/account.repository";
import type { ISessionRepository } from "../../domain/repositories/session.repository";
import type { AuthService } from "../../domain/services/auth.service";
import type { UsernameGeneratorService } from "../../domain/services/username-generator.service";

interface SignInUseCaseDependencies extends UseCaseDependencies {
	usernameGenerator: UsernameGeneratorService;
	accountRepository: IAccountRepository;
	sessionRepository: ISessionRepository;
	userRepository: IUserRepository;
	authService: AuthService;
}

export class SignInUseCase extends TransactionalUseCase<
	SignInDto,
	AuthResponseDto,
	AuthMetadata
> {
	constructor(private readonly deps: SignInUseCaseDependencies) {
		super(deps); // O pai (TransactionalUseCase) já guarda o unitOfWork e logger
	}

	async execute(
		data: SignInDto,
		metadata: AuthMetadata,
	): Promise<AuthResponseDto> {
		this.logger.info(
			{ identifier: data.identifier },
			"Iniciando tentativa de login",
		);

		const { user, session, accessToken, expiresAt } = await this.unitOfWork.run(
			async () => {
				// 1. Validações de Credenciais
				const user = await this.validateUserExists(data.identifier);
				this.validateEmailVerified(user);

				const account = await this.validateAccountExists(user);
				await this.validatePassword(account, data.password);

				// 2. Criação da Sessão
				const { session, accessToken, expiresAt } =
					await this.deps.authService.createAuthenticatedSession(
						user,
						metadata,
					);

				await this.deps.sessionRepository.save(session);

				return { user, session, accessToken, expiresAt };
			},
		);

		// 3. Notificação (Fora da transação para não travar o banco)
		await this.deps.authService.notifyAuthentication("login", user, session);

		return this.buildResponse(user, accessToken, expiresAt);
	}

	// --- Métodos Auxiliares ---

	private async validateUserExists(identifier: string) {
		const user = await this.deps.userRepository.findByIdentifier(identifier);
		if (!user) throw new UnauthorizedError("Credenciais inválidas");
		return user;
	}

	private validateEmailVerified(user: User) {
		if (!user.isEmailVerified()) {
			throw new UnauthorizedError("E-mail não verificado");
		}
	}

	private async validateAccountExists(user: User) {
		const account = await this.deps.accountRepository.findByProviderAccount(
			"local",
			user.getId(),
		);

		if (!(account instanceof LocalAccount)) {
			throw new UnauthorizedError("Esta conta não possui autenticação local");
		}
		return account;
	}

	private async validatePassword(account: LocalAccount, password: string) {
		const isValid = await account.verifyPassword(password);
		if (!isValid) throw new UnauthorizedError("Credenciais inválidas");
	}

	private buildResponse(user: User, accessToken: string, _expiresAt: Date) {
		return {
			session: {
				token: accessToken,
			},
			user: user.toDTO(),
		};
	}
}
\n\n===== FILE: ./src/modules/auth/application/use-cases/get-session.use-case.ts =====
import type {
	GetSessionDto,
	GetSessionResponseDto,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { UnauthorizedError } from "../../../../core/errors";
import { env } from "../../../../shared/config";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import type { ISessionRepository } from "../../domain/repositories/session.repository";
import type { JwtTokenService } from "../../domain/services/jwt.service";

interface GetSessionUseCaseDependencies extends UseCaseDependencies {
	sessionRepository: ISessionRepository;
	userRepository: IUserRepository;
	jwtService: JwtTokenService;
}

export class GetSessionUseCase extends TransactionalUseCase<
	GetSessionDto,
	GetSessionResponseDto
> {
	constructor(private readonly deps: GetSessionUseCaseDependencies) {
		super(deps); // O pai (TransactionalUseCase) já guarda o unitOfWork e logger
	}

	async execute(data: GetSessionDto): Promise<GetSessionResponseDto> {
		const now = Date.now();
		// 1. Decodifica e Valida a assinatura do JWT
		const decoded = await this.deps.jwtService.verifyToken(data.token);

		// 2. Busca a sessão no banco usando o ID (sid) que estava dentro do JWT
		const session = await this.deps.sessionRepository.findOneById(decoded.sid);

		// 3. Validações de Banco
		if (!session) {
			// Se o token é válido mas não tá no banco, foi revogado/logout
			throw new UnauthorizedError("Sessão revogada ou inválida.");
		}

		if (session.isExpired()) {
			// Se passou da data no banco (mesmo que o JWT esteja válido)
			throw new UnauthorizedError("Sessão expirada.");
		}

		const expiresAtMs = session.getExpiresAt().getTime();

		const REFRESH_THRESHOLD_MS =
			env.SESSION_REFRESH_THRESHOLD_DAYS * 24 * 60 * 60 * 1000;
		const SESSION_LIFE_MS = env.SESSION_EXPIRATION_DAYS * 24 * 60 * 60 * 1000;

		const timeUntilExpiry = expiresAtMs - now;

		let currentToken = data.token;

		if (timeUntilExpiry < REFRESH_THRESHOLD_MS) {
			session.extend(SESSION_LIFE_MS);
			await this.deps.sessionRepository.save(session);

			const { accessToken } = await this.deps.jwtService.generateToken(
				session.getUserId(),
				session.getId(),
				session.getExpiresAt(),
			);
			currentToken = accessToken;

			this.logger.info(
				{ sessionId: session.getId() },
				"Sessão renovada automaticamente",
			);
		}

		const user = await this.deps.userRepository.findById(session.getUserId());

		if (!user || user.isBanned()) {
			throw new UnauthorizedError("Usuário inválido.");
		}

		return {
			session: session.toDTO(currentToken),
			user: user.toDTO(),
		};
	}
}
\n\n===== FILE: ./src/modules/auth/auth.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asClass, Lifetime } from "awilix";
import type { FastifyInstance } from "fastify";
import { AsyncTask, SimpleIntervalJob } from "toad-scheduler";
import type { Module } from "../../shared/module/module.interface";
import { ForgotPasswordUseCase } from "./application/use-cases/forgot-password.use-case";
import { GetSessionUseCase } from "./application/use-cases/get-session.use-case";
import { ListSessionsUseCase } from "./application/use-cases/list-sessions.use-case";
import { ResetPasswordUseCase } from "./application/use-cases/reset-password.use-case";
import { RevokeAllSessionsUseCase } from "./application/use-cases/revoke-all-sessions.use-case";
import { RevokeOtherSessionsUseCase } from "./application/use-cases/revoke-other-sessions.use-case";
import { RevokeSessionUseCase } from "./application/use-cases/revoke-session.use-case";
import { SignInUseCase } from "./application/use-cases/sign-in.use-case";
import { SignOutUseCase } from "./application/use-cases/sign-out.use-case";
import { SignUpUseCase } from "./application/use-cases/sign-up.use-case";
import { SendVerificationCodeUseCase } from "./application/use-cases/verification/send-verification-code.use-case";
import { VerifyCodeUseCase } from "./application/use-cases/verification/verify-code.use-case";
import { AuthService } from "./domain/services/auth.service";
import { JwtTokenService } from "./domain/services/jwt.service";
import { UsernameGeneratorService } from "./domain/services/username-generator.service";
import { AccountRepositoryImpl } from "./infrastructure/database/drizzle/repositories/account.repository.impl";
import { SessionRepositoryImpl } from "./infrastructure/database/drizzle/repositories/session.repository.impl";
import { VerificationRepositoryImpl } from "./infrastructure/database/drizzle/repositories/verification.repository.impl";
import { authRoutes } from "./infrastructure/http/routes/auth.routes";
import { SessionCleanupTask } from "./infrastructure/jobs/session.cleanup";
import { VerificationCleanupTask } from "./infrastructure/jobs/verification.cleanup";

export class AuthModule implements Module {
	async register(app: FastifyInstance): Promise<void> {
		diContainer.register({
			// --- REPOSITORIES (SCOPED) ---
			// Eles devem ser Scoped porque dependem do UnitOfWork/Logger do request
			accountRepository: asClass(AccountRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			sessionRepository: asClass(SessionRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			verificationRepository: asClass(VerificationRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),

			// --- DOMAIN SERVICES (SINGLETON se forem puros, SCOPED se usarem Repo) ---
			usernameGenerator: asClass(UsernameGeneratorService, {
				lifetime: Lifetime.SINGLETON,
			}),
			authService: asClass(AuthService, { lifetime: Lifetime.SCOPED }), // Se usa repositório, vira Scoped!
			jwtService: asClass(JwtTokenService, { lifetime: Lifetime.SINGLETON }),

			// --- USE CASES (SCOPED) ---
			signUpUseCase: asClass(SignUpUseCase, { lifetime: Lifetime.SCOPED }),
			signInUseCase: asClass(SignInUseCase, { lifetime: Lifetime.SCOPED }),
			signOutUseCase: asClass(SignOutUseCase, { lifetime: Lifetime.SCOPED }),
			sendVerificationCodeUseCase: asClass(SendVerificationCodeUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			verifyCodeUseCase: asClass(VerifyCodeUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			forgotPasswordUseCase: asClass(ForgotPasswordUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			resetPasswordUseCase: asClass(ResetPasswordUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			getSessionUseCase: asClass(GetSessionUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			revokeSessionUseCase: asClass(RevokeSessionUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			revokeOtherSessionsUseCase: asClass(RevokeOtherSessionsUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			revokeAllSessionsUseCase: asClass(RevokeAllSessionsUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			listSessionsUseCase: asClass(ListSessionsUseCase, {
				lifetime: Lifetime.SCOPED,
			}),

			// --- JOBS (SCOPED) ---
			verificationCleanupTask: asClass(VerificationCleanupTask, {
				lifetime: Lifetime.SINGLETON,
			}),
			sessionCleanupTask: asClass(SessionCleanupTask, {
				lifetime: Lifetime.SINGLETON,
			}),
		});
		// Rotas
		await app.register(authRoutes, { prefix: "/v1/auth" });
	}
	async bootstrap(app: FastifyInstance) {
		// 1. Pega as tasks do container global
		const { sessionCleanupTask, verificationCleanupTask } =
			app.diContainer.cradle;

		// 2. Define os jobs
		const sessionJob = new SimpleIntervalJob(
			{ minutes: 30 },
			new AsyncTask("auth:session_cleanup", () =>
				sessionCleanupTask.handleCron(),
			),
			{ id: "auth_session_01", preventOverrun: true },
		);

		const verificationJob = new SimpleIntervalJob(
			{ hours: 1 },
			new AsyncTask("auth:verification_cleanup", () =>
				verificationCleanupTask.handleCron(),
			),
			{ id: "auth_verification_01", preventOverrun: true },
		);

		// 3. Agenda no scheduler do app
		app.scheduler.addSimpleIntervalJob(sessionJob);
		app.scheduler.addSimpleIntervalJob(verificationJob);

		app.log.info("📅 AuthModule: Cron jobs agendados");
	}
}
\n\n===== FILE: ./src/modules/user/infrastructure/database/drizzle/repositories/user.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { users } from "../../../../../../infrastructure/database/drizzle/schema";
import { User } from "../../../../domain/entities/user.entity";
import type { IUserRepository } from "../../../../domain/repositories/user.repository";
import { EmailVO } from "../../../../domain/value-objects/email.vo";
import { UsernameVO } from "../../../../domain/value-objects/username.vo";

export class UserRepositoryImpl
	extends BaseRepository
	implements IUserRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// Queries
	async findById(id: string): Promise<User | null> {
		const result = await this.db.query.users.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByEmail(email: string): Promise<User | null> {
		const result = await this.db.query.users.findFirst({
			where: {
				email,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUsername(username: string): Promise<User | null> {
		const result = await this.db.query.users.findFirst({
			where: {
				username,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByIdentifier(identifier: string): Promise<User | null> {
		const result = await this.db.query.users.findFirst({
			where: {
				OR: [{ email: identifier }, { username: identifier }],
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// Commands
	async save(user: User): Promise<void> {
		const data = this.toPersistence(user);

		await this.db
			.insert(users)
			.values(data)
			.onConflictDoUpdate({
				target: users.id,
				set: {
					name: data.name,
					image: data.image,
					emailVerified: data.emailVerified,
					updatedAt: new Date(),
				},
			});
	}

	async softDelete(userId: string): Promise<void> {
		await this.db
			.update(users)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(users.id, userId));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(users)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(users.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(users).where(eq(users.id, id));
	}

	// Mappers
	private toDomain(raw: typeof users.$inferSelect): User {
		return new User(
			raw.id,
			UsernameVO.create(raw.username),
			raw.name,
			EmailVO.create(raw.email),
			raw.role,
			raw.emailVerified,
			raw.image ?? null,
			null,
			raw.banned ?? false,
			raw.banReason ?? null,
			raw.banExpires ?? null,
			raw.createdAt,
			raw.updatedAt,
			raw.deletedAt ?? undefined,
		);
	}

	private toPersistence(user: User): typeof users.$inferInsert {
		return {
			id: user.getId(),
			username: user.getUsername().getValue(),
			name: user.getName(),
			email: user.getEmail().getValue(),
			role: user.getRole(),
			emailVerified: user.isEmailVerified(),
			image: user.getImage() ?? null,
			phone: null,
			banned: user.isBanned?.() ?? false,
			banReason: user.getBanReason?.() ?? null,
			banExpires: user.getBanExpires?.() ?? null,
			createdAt: user.getCreatedAt(),
			updatedAt: user.getUpdatedAt(),
			deletedAt: user.getDeletedAt() ?? null,
		};
	}
}
\n\n===== FILE: ./src/modules/user/domain/repositories/user.repository.ts =====
import type { User } from "../entities/user.entity";

export interface IUserRepository {
	findById(id: string): Promise<User | null>;
	findByEmail(email: string): Promise<User | null>;
	findByUsername(username: string): Promise<User | null>;
	findByIdentifier(identifier: string): Promise<User | null>;

	save(user: User): Promise<void>;
	softDelete(userId: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/user/domain/value-objects/username.vo.ts =====
export class UsernameVO {
	private constructor(private readonly value: string) {}

	static create(value: string): UsernameVO {
		if (!value || value.length < 3 || value.length > 30) {
			throw new Error("Username deve ter entre 3 e 30 caracteres");
		}

		if (!/^[a-zA-Z0-9_-]+$/.test(value)) {
			throw new Error(
				"Username deve conter apenas letras, números e underscore",
			);
		}
		return new UsernameVO(value.toLowerCase()); // normaliza
	}

	getValue(): string {
		return this.value;
	}

	equals(other: UsernameVO): boolean {
		return this.value === other.value;
	}
}
\n\n===== FILE: ./src/modules/user/domain/value-objects/email.vo.ts =====
export class EmailVO {
	private constructor(private readonly value: string) {}

	static create(value: string): EmailVO {
		if (!value || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
			throw new Error("Email inválido");
		}
		return new EmailVO(value.toLowerCase());
	}

	getValue(): string {
		return this.value;
	}

	equals(other: EmailVO): boolean {
		return this.value === other.value;
	}
}
\n\n===== FILE: ./src/modules/user/domain/entities/user.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { UserRole } from "../enums/user-role";
import type { EmailVO } from "../value-objects/email.vo";
import type { UsernameVO } from "../value-objects/username.vo";

interface ProfileUpdateData {
	name?: string;
	image?: string;
}

export class User {
	constructor(
		private readonly id: string,
		private username: UsernameVO,
		private name: string,
		private email: EmailVO,
		private role: UserRole,
		private emailVerified: boolean = false,
		private image: string | null = null,
		private displayUsername: string | null = null,
		private banned: boolean = false,
		private banReason: string | null = null,
		private banExpires: Date | null = null,
		private readonly createdAt: Date = new Date(),
		private updatedAt: Date = new Date(),
		private deletedAt: Date | null = null,
	) {}

	// Getters

	getId(): string {
		return this.id;
	}

	getName(): string {
		return this.name;
	}

	getEmail(): EmailVO {
		return this.email;
	}

	getUsername(): UsernameVO {
		return this.username;
	}

	getImage(): string | null {
		return this.image;
	}

	getRole(): UserRole {
		return this.role;
	}

	getBanReason(): string | null {
		return this.banReason;
	}

	getBanExpires(): Date | null {
		return this.banExpires;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	getDeletedAt(): Date | null {
		return this.deletedAt;
	}

	isEmailVerified(): boolean {
		return this.emailVerified;
	}

	isDeleted(): boolean {
		return this.deletedAt !== null;
	}

	isBanned(): boolean {
		return this.banned;
	}

	// Commands
	changeEmail(newEmail: EmailVO): void {
		if (this.email.equals(newEmail)) {
			throw new BusinessError(
				"O novo e-mail não pode ser igual ao e-mail atual.",
				"SAME_EMAIL_ERROR",
			);
		}
		this.email = newEmail;
		this.emailVerified = false;
		this.updatedAt = new Date();
	}

	changeUsername(newUsername: UsernameVO): void {
		if (this.username.equals(newUsername)) {
			throw new BusinessError(
				"O novo username não pode ser igual ao username atual.",
				"SAME_USERNAME_ERROR",
			);
		}
		this.username = newUsername;
		this.updatedAt = new Date();
	}

	verifyEmail(): void {
		if (this.emailVerified) {
			throw new BusinessError(
				"Este email já está verificado",
				"EMAIL_ALREADY_VERIFIED",
			);
		}
		this.emailVerified = true;
		this.updatedAt = new Date();
	}

	public promoteToAdmin(): void {
		if (this.role === "admin") {
			throw new BusinessError(
				"Usuário já é um administrador",
				"USER_ALREADY_IS_ADMIN",
			);
		}
		this.role = "admin";
		this.updatedAt = new Date();
	}

	ban(reason: string, expires?: Date): void {
		if (this.role === "admin") {
			throw new BusinessError(
				"Administradores não podem ser banidos automaticamente.",
				"CANNOT_BAN_ADMIN",
			);
		}
		this.banned = true;
		this.banReason = reason;
		this.banExpires = expires ?? null;
		this.updatedAt = new Date();
	}

	unban(): void {
		this.banned = false;
		this.banReason = null;
		this.banExpires = null;
		this.updatedAt = new Date();
	}

	updateDisplayUsername(newDisplayName: string | null): void {
		this.displayUsername = newDisplayName;
		this.updatedAt = new Date();
	}

	updateProfile(data: ProfileUpdateData): void {
		let changed = false;

		if (data.name !== undefined) {
			if (!data.name || data.name.trim().length === 0) {
				throw new Error("Name cannot be empty");
			}
			if (data.name.trim() !== this.name) {
				this.name = data.name.trim();
				changed = true;
			}
		}

		if (data.image !== undefined && data.image !== this.image) {
			this.image = data.image;
			changed = true;
		}

		if (changed) {
			this.updatedAt = new Date();
		}
	}

	softDelete(): void {
		if (this.deletedAt) {
			throw new BusinessError(
				"O usuário já está excluído.",
				"USER_ALREADY_DELETED",
			);
		}
		this.deletedAt = new Date();
		this.updatedAt = new Date();
	}

	restore(): void {
		if (!this.deletedAt) {
			throw new BusinessError(
				"O usuário não foi excluído.",
				"USER_NOT_IS_DELETED",
			);
		}
		this.deletedAt = null;
		this.updatedAt = new Date();
	}

	toDTO() {
		return {
			id: this.id,
			username: this.username.getValue(),
			name: this.name,
			role: this.role,
			email: this.email.getValue(),
			emailVerified: this.emailVerified,
			image: this.image || null,
			displayUsername: this.displayUsername ?? null,
			banned: this.banned,
			banReason: this.banReason,
			banExpires: this.banExpires,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
			deletedAt: this.deletedAt ?? null,
		};
	}
}
\n\n===== FILE: ./src/modules/user/domain/enums/user-role.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { userRoles } from "../../../../infrastructure/database/drizzle/schema/enums";

export const userRolesSchema = createSelectSchema(userRoles);

export type UserRole = z.infer<typeof userRolesSchema>;
\n\n===== FILE: ./src/modules/user/user.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asClass, Lifetime } from "awilix";
import type { FastifyInstance } from "fastify";
import type { Module } from "../../shared/module/module.interface";
import { UserRepositoryImpl } from "./infrastructure/database/drizzle/repositories/user.repository.impl";

export class UserModule implements Module {
	async register(app: FastifyInstance): Promise<void> {
		diContainer.register({
			// Repositories
			userRepository: asClass(UserRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
		});

		app.log.info("✅ UserModule registered");
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/base-profile.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { ProfileStatus } from "../enums/profile-status";

/**
 * BaseProfile - Classe base para TODOS os perfis
 *
 * NÃO existe mais distinção entre "pago" e "gratuito" nas entidades.
 * O que determina funcionalidades é a SUBSCRIPTION (plano), não o perfil.
 *
 * Usado por:
 * - ResidentProfile
 * - SyndicProfile
 * - EnterpriseProfile
 * - LocalCompanyProfile
 * - MarketingProfile
 */
export abstract class BaseProfile {
	constructor(
		protected readonly id: string,
		protected readonly userId: string,
		protected status: ProfileStatus,
		protected readonly createdAt: Date,
		protected updatedAt: Date,
		protected deletedAt: Date | null = null,
	) {}

	// ==================== GETTERS ====================
	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getStatus(): ProfileStatus {
		return this.status;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	getDeletedAt(): Date | null {
		return this.deletedAt;
	}

	// ==================== STATUS CHECKS ====================
	isActive(): boolean {
		return this.status === "active";
	}

	isIncomplete(): boolean {
		return this.status === "incomplete";
	}

	isPendingPayment(): boolean {
		return this.status === "pending_payment";
	}

	isDeleted(): boolean {
		return this.deletedAt !== null;
	}

	// ==================== COMMANDS ====================
	markAsPendingPayment(): void {
		if (this.status === "pending_payment") {
			throw new BusinessError(
				"Perfil já está aguardando pagamento.",
				"PROFILE_ALREADY_PENDING_PAYMENT",
			);
		}
		if (this.status === "active") {
			throw new BusinessError(
				"Perfil já está ativo, não pode voltar para pendente.",
				"PROFILE_ALREADY_ACTIVE",
			);
		}
		this.status = "pending_payment";
		this.updatedAt = new Date();
	}

	activate(): void {
		if (this.status === "active") {
			throw new BusinessError(
				"Perfil já está ativo.",
				"PROFILE_ALREADY_ACTIVE",
			);
		}
		this.status = "active";
		this.updatedAt = new Date();
	}

	softDelete(): void {
		if (this.deletedAt) {
			throw new BusinessError(
				"Perfil já está excluído.",
				"PROFILE_ALREADY_DELETED",
			);
		}
		this.deletedAt = new Date();
		this.updatedAt = new Date();
	}

	restore(): void {
		if (!this.deletedAt) {
			throw new BusinessError(
				"Perfil não foi excluído.",
				"PROFILE_NOT_IS_DELETED",
			);
		}
		this.deletedAt = null;
		this.updatedAt = new Date();
	}

	// ==================== BASE DTO ====================
	protected toBaseDto() {
		return {
			id: this.id,
			userId: this.userId,
			status: this.status,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
			deletedAt: this.deletedAt,
		};
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/resident.entity.ts =====
import type { ProfileStatus } from "../enums/profile-status";
import type { CpfVO } from "../value-objects/cpf.vo";
import { BaseProfile } from "./base-profile.entity";

interface ResidentAddress {
	street: string;
	number: string;
	block: string | null;
	district: string;
	city: string;
	state: string;
	zipCode: string;
}

interface ResidentProfileProps {
	id: string;
	userId: string;
	birthDate: Date;
	cpf: CpfVO;
	address: ResidentAddress;
	buildingNameOrCode: string | null;
	status: ProfileStatus;
	createdAt: Date;
	updatedAt: Date;
	deletedAt?: Date | null;
}

export class ResidentProfile extends BaseProfile {
	private birthDate: Date;
	private cpf: CpfVO;
	private address: ResidentAddress;
	private buildingNameOrCode: string | null;

	constructor(props: ResidentProfileProps) {
		super(
			props.id,
			props.userId,
			props.status,
			props.createdAt,
			props.updatedAt,
			props.deletedAt ?? null,
		);
		this.birthDate = props.birthDate;
		this.cpf = props.cpf;
		this.address = props.address;
		this.buildingNameOrCode = props.buildingNameOrCode;
	}

	getBirthDate(): Date {
		return this.birthDate;
	}

	getCpf(): string {
		return this.cpf.getValue();
	}

	getAddress(): ResidentAddress {
		return { ...this.address };
	}

	getBuildingNameOrCode(): string | null {
		return this.buildingNameOrCode;
	}

	updateAddress(address: Partial<ResidentAddress>): void {
		this.address = { ...this.address, ...address };
		this.updatedAt = new Date();
	}

	updateBuildingLink(buildingNameOrCode: string | null): void {
		this.buildingNameOrCode = buildingNameOrCode;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			...super.toBaseDto(),
			birthDate: this.birthDate,
			cpf: this.cpf.getValue(),
			street: this.address.street,
			number: this.address.number,
			block: this.address.block,
			district: this.address.district,
			city: this.address.city,
			state: this.address.state,
			zipCode: this.address.zipCode,
			buildingNameOrCode: this.buildingNameOrCode,
		};
	}

	static create(props: ResidentProfileProps): ResidentProfile {
		return new ResidentProfile(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		birthDate: Date;
		cpf: string;
		street: string;
		number: string;
		block: string | null;
		district: string;
		city: string;
		state: string;
		zipCode: string;
		buildingNameOrCode: string | null;
		status: ProfileStatus;
		createdAt: Date;
		updatedAt: Date;
		deletedAt?: Date | null;
	}): ResidentProfile {
		const { CpfVO } = require("../value-objects/cpf.vo");
		const cpfVO = CpfVO.create(data.cpf);

		return new ResidentProfile({
			id: data.id,
			userId: data.userId,
			birthDate: data.birthDate,
			cpf: cpfVO,
			address: {
				street: data.street,
				number: data.number,
				block: data.block,
				district: data.district,
				city: data.city,
				state: data.state,
				zipCode: data.zipCode,
			},
			buildingNameOrCode: data.buildingNameOrCode,
			status: data.status,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
			deletedAt: data.deletedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/syndic.entity.ts =====
import type { ProfileStatus } from "../enums/profile-status";
import type { SyndicType } from "../enums/syndic-type";
import type { CnpjVO } from "../value-objects";
import type { CpfVO } from "../value-objects/cpf.vo";
import { BaseProfile } from "./base-profile.entity";

interface SyndicAddress {
	street: string;
	number: string;
	block: string | null;
	district: string;
	city: string;
	state: string;
	zipCode: string;
}

interface SyndicCertifications {
	hasAbracsMember: boolean;
	hasCivilLiabilityInsurance: boolean;
	hasLegalAdvice: boolean;
	hasAccountingAdvice: boolean;
	hasWorkSafetyAdvice: boolean;
	hasLgpdCompliance: boolean;
	hasNr1Compliance: boolean;
	hasComplianceProgram: boolean;
	hasReclameAquiSeal: boolean;
	otherCertifications: string | null;
	awards: string | null;
}

interface SyndicProfileProps {
	id: string;
	userId: string;
	birthDate: Date;
	document: CpfVO | CnpjVO;
	address: SyndicAddress;
	type: SyndicType;
	experienceYears: number;
	buildingsCount: number;
	operatingCities: string[] | null;
	certifications: SyndicCertifications;
	miniBio: string | null;
	educationAndCertifications: string | null;
	linkedAdministrators: string | null;
	status: ProfileStatus;
	createdAt: Date;
	updatedAt: Date;
	deletedAt?: Date | null;
}

export class SyndicProfile extends BaseProfile {
	private birthDate: Date;
	private document: CpfVO | CnpjVO;
	private address: SyndicAddress;
	private type: SyndicType;
	private experienceYears: number;
	private buildingsCount: number;
	private operatingCities: string[] | null;
	private certifications: SyndicCertifications;
	private miniBio: string | null;
	private educationAndCertifications: string | null;
	private linkedAdministrators: string | null;

	constructor(props: SyndicProfileProps) {
		super(
			props.id,
			props.userId,
			props.status,
			props.createdAt,
			props.updatedAt,
			props.deletedAt ?? null,
		);
		this.birthDate = props.birthDate;
		this.document = props.document;
		this.address = props.address;
		this.type = props.type;
		this.experienceYears = props.experienceYears;
		this.buildingsCount = props.buildingsCount;
		this.operatingCities = props.operatingCities;
		this.certifications = props.certifications;
		this.miniBio = props.miniBio;
		this.educationAndCertifications = props.educationAndCertifications;
		this.linkedAdministrators = props.linkedAdministrators;
	}

	getBirthDate(): Date {
		return this.birthDate;
	}

	getDocument(): string {
		return this.document.getValue();
	}

	getAddress(): SyndicAddress {
		return { ...this.address };
	}

	getType(): SyndicType {
		return this.type;
	}

	getExperienceYears(): number {
		return this.experienceYears;
	}

	getBuildingsCount(): number {
		return this.buildingsCount;
	}

	getOperatingCities(): string[] | null {
		return this.operatingCities ? [...this.operatingCities] : null;
	}

	getCertifications(): SyndicCertifications {
		return { ...this.certifications };
	}

	getMiniBio(): string | null {
		return this.miniBio;
	}

	getEducationAndCertifications(): string | null {
		return this.educationAndCertifications;
	}

	getLinkedAdministrators(): string | null {
		return this.linkedAdministrators;
	}

	isProfessional(): boolean {
		return this.type === "profissional";
	}

	isResident(): boolean {
		return this.type === "morador";
	}

	updateAddress(address: Partial<SyndicAddress>): void {
		this.address = { ...this.address, ...address };
		this.updatedAt = new Date();
	}

	updateExperience(experienceYears: number, buildingsCount: number): void {
		this.experienceYears = experienceYears;
		this.buildingsCount = buildingsCount;
		this.updatedAt = new Date();
	}

	updateOperatingCities(cities: string[]): void {
		this.operatingCities = cities;
		this.updatedAt = new Date();
	}

	updateCertifications(certifications: Partial<SyndicCertifications>): void {
		this.certifications = { ...this.certifications, ...certifications };
		this.updatedAt = new Date();
	}

	updateMiniBio(miniBio: string | null): void {
		this.miniBio = miniBio;
		this.updatedAt = new Date();
	}

	updateEducationAndCertifications(text: string | null): void {
		this.educationAndCertifications = text;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			...this.toBaseDto(),
			birthDate: this.birthDate,
			document: this.document.getValue(),
			street: this.address.street,
			number: this.address.number,
			block: this.address.block,
			district: this.address.district,
			city: this.address.city,
			state: this.address.state,
			zipCode: this.address.zipCode,
			type: this.type,
			experienceYears: this.experienceYears,
			buildingsCount: this.buildingsCount,
			operatingCities: this.operatingCities,
			hasAbracsMember: this.certifications.hasAbracsMember,
			hasCivilLiabilityInsurance:
				this.certifications.hasCivilLiabilityInsurance,
			hasLegalAdvice: this.certifications.hasLegalAdvice,
			hasAccountingAdvice: this.certifications.hasAccountingAdvice,
			hasWorkSafetyAdvice: this.certifications.hasWorkSafetyAdvice,
			hasLgpdCompliance: this.certifications.hasLgpdCompliance,
			hasNr1Compliance: this.certifications.hasNr1Compliance,
			hasComplianceProgram: this.certifications.hasComplianceProgram,
			hasReclameAquiSeal: this.certifications.hasReclameAquiSeal,
			otherCertifications: this.certifications.otherCertifications,
			awards: this.certifications.awards,
			miniBio: this.miniBio,
			educationAndCertifications: this.educationAndCertifications,
			linkedAdministrators: this.linkedAdministrators,
		};
	}

	static create(props: SyndicProfileProps): SyndicProfile {
		return new SyndicProfile(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		birthDate: Date;
		document: string;
		street: string;
		number: string;
		block: string | null;
		district: string;
		city: string;
		state: string;
		zipCode: string;
		type: SyndicType;
		experienceYears: number;
		buildingsCount: number;
		operatingCities: string[] | null;
		hasAbracsMember: boolean | null;
		hasCivilLiabilityInsurance: boolean | null;
		hasLegalAdvice: boolean | null;
		hasAccountingAdvice: boolean | null;
		hasWorkSafetyAdvice: boolean | null;
		hasLgpdCompliance: boolean | null;
		hasNr1Compliance: boolean | null;
		hasComplianceProgram: boolean | null;
		hasReclameAquiSeal: boolean | null;
		otherCertifications: string | null;
		awards: string | null;
		miniBio: string | null;
		educationAndCertifications: string | null;
		linkedAdministrators: string | null;
		status: ProfileStatus;
		createdAt: Date;
		updatedAt: Date;
		deletedAt?: Date | null;
	}): SyndicProfile {
		const { CpfVO } = require("../value-objects/cpf.vo");
		const { CnpjVO } = require("../value-objects/cnpj.vo");

		// Determina se é CPF ou CNPJ pelo comprimento
		const cleanDocument = data.document.replace(/\D/g, "");
		const documentVO =
			cleanDocument.length === 11
				? CpfVO.create(data.document)
				: CnpjVO.create(data.document);

		return new SyndicProfile({
			id: data.id,
			userId: data.userId,
			birthDate: data.birthDate,
			document: documentVO,
			address: {
				street: data.street,
				number: data.number,
				block: data.block,
				district: data.district,
				city: data.city,
				state: data.state,
				zipCode: data.zipCode,
			},
			type: data.type,
			experienceYears: data.experienceYears,
			buildingsCount: data.buildingsCount,
			operatingCities: data.operatingCities,
			certifications: {
				hasAbracsMember: data.hasAbracsMember ?? false,
				hasCivilLiabilityInsurance: data.hasCivilLiabilityInsurance ?? false,
				hasLegalAdvice: data.hasLegalAdvice ?? false,
				hasAccountingAdvice: data.hasAccountingAdvice ?? false,
				hasWorkSafetyAdvice: data.hasWorkSafetyAdvice ?? false,
				hasLgpdCompliance: data.hasLgpdCompliance ?? false,
				hasNr1Compliance: data.hasNr1Compliance ?? false,
				hasComplianceProgram: data.hasComplianceProgram ?? false,
				hasReclameAquiSeal: data.hasReclameAquiSeal ?? false,
				otherCertifications: data.otherCertifications,
				awards: data.awards,
			},
			miniBio: data.miniBio,
			educationAndCertifications: data.educationAndCertifications,
			linkedAdministrators: data.linkedAdministrators,
			status: data.status,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
			deletedAt: data.deletedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/enterprise.entity.ts =====
import type { ProfileStatus } from "../enums/profile-status";
import type { CnpjVO } from "../value-objects/cnpj.vo";
import { BaseProfile } from "./base-profile.entity";

interface EnterpriseAddress {
	street: string;
	number: string;
	block: string | null;
	district: string;
	city: string;
	state: string;
	zipCode: string;
}

interface LegalRepresentative {
	name: string;
	email: string;
	phone?: string;
}

interface CommercialContact {
	email: string;
	phone: string;
}

interface FinanceContact {
	name: string;
	email: string;
	phone: string;
}

interface EnterpriseCertifications {
	hasCivilLiabilityInsurance: boolean;
	hasLegalAdvice: boolean;
	hasAccountingAdvice: boolean;
	hasWorkSafetyAdvice: boolean;
	hasTechnicalManager: boolean;
	hasProfessionalCouncilRegularity: boolean;
	hasLgpdCompliance: boolean;
	hasNr1Compliance: boolean;
	hasComplianceProgram: boolean;
}

interface EnterpriseIsos {
	hasIso9001: boolean;
	hasIso14001: boolean;
	hasIso45001: boolean;
	hasIso37001: boolean;
	hasIso19600_37301: boolean;
}

interface EnterpriseSeals {
	hasEsg: boolean;
	hasGreenSeal: boolean;
	hasChildFriendlySeal: boolean;
	hasCarbonFreeSeal: boolean;
	hasEuRecicloSeal: boolean;
	hasReclameAquiSeal: boolean;
	hasCipa: boolean;
}

interface EnterpriseProfileProps {
	id: string;
	userId: string;
	companyName: string;
	companyTradeName: string;
	cnpj: CnpjVO;
	foundationDate: Date;
	legalRepresentative: LegalRepresentative;
	commercialContact: CommercialContact;
	address: EnterpriseAddress;
	financeContact: FinanceContact;
	operatingCities: string[] | null;
	certifications: EnterpriseCertifications;
	isos: EnterpriseIsos;
	seals: EnterpriseSeals;
	nrs: string | null;
	otherCertifications: string | null;
	ownCertifications: string | null;
	awards: string | null;
	institutionalDescription: string | null;
	portfolioUrl: string | null;
	status: ProfileStatus;
	createdAt: Date;
	updatedAt: Date;
	deletedAt?: Date | null;
}

export class EnterpriseProfile extends BaseProfile {
	private companyName: string;
	private companyTradeName: string;
	private cnpj: CnpjVO;
	private foundationDate: Date;
	private legalRepresentative: LegalRepresentative;
	private commercialContact: CommercialContact;
	private address: EnterpriseAddress;
	private financeContact: FinanceContact;
	private operatingCities: string[] | null;
	private certifications: EnterpriseCertifications;
	private isos: EnterpriseIsos;
	private seals: EnterpriseSeals;
	private nrs: string | null;
	private otherCertifications: string | null;
	private ownCertifications: string | null;
	private awards: string | null;
	private institutionalDescription: string | null;
	private portfolioUrl: string | null;

	constructor(props: EnterpriseProfileProps) {
		super(
			props.id,
			props.userId,
			props.status,
			props.createdAt,
			props.updatedAt,
			props.deletedAt ?? null,
		);
		this.companyName = props.companyName;
		this.companyTradeName = props.companyTradeName;
		this.cnpj = props.cnpj;
		this.foundationDate = props.foundationDate;
		this.legalRepresentative = props.legalRepresentative;
		this.commercialContact = props.commercialContact;
		this.address = props.address;
		this.financeContact = props.financeContact;
		this.operatingCities = props.operatingCities;
		this.certifications = props.certifications;
		this.isos = props.isos;
		this.seals = props.seals;
		this.nrs = props.nrs;
		this.otherCertifications = props.otherCertifications;
		this.ownCertifications = props.ownCertifications;
		this.awards = props.awards;
		this.institutionalDescription = props.institutionalDescription;
		this.portfolioUrl = props.portfolioUrl;
	}

	getCompanyName(): string {
		return this.companyName;
	}

	getCompanyTradeName(): string {
		return this.companyTradeName;
	}

	getCnpj(): string {
		return this.cnpj.getValue();
	}

	getFoundationDate(): Date {
		return this.foundationDate;
	}

	getLegalRepresentative(): LegalRepresentative {
		return { ...this.legalRepresentative };
	}

	getCommercialContact(): CommercialContact {
		return { ...this.commercialContact };
	}

	getAddress(): EnterpriseAddress {
		return { ...this.address };
	}

	getFinanceContact(): FinanceContact {
		return { ...this.financeContact };
	}

	getOperatingCities(): string[] | null {
		return this.operatingCities ? [...this.operatingCities] : null;
	}

	getCertifications(): EnterpriseCertifications {
		return { ...this.certifications };
	}

	getIsos(): EnterpriseIsos {
		return { ...this.isos };
	}

	getSeals(): EnterpriseSeals {
		return { ...this.seals };
	}

	getNrs(): string | null {
		return this.nrs;
	}

	getOtherCertifications(): string | null {
		return this.otherCertifications;
	}

	getOwnCertifications(): string | null {
		return this.ownCertifications;
	}

	getAwards(): string | null {
		return this.awards;
	}

	getInstitutionalDescription(): string | null {
		return this.institutionalDescription;
	}

	getPortfolioUrl(): string | null {
		return this.portfolioUrl;
	}

	updateLegalRepresentative(data: Partial<LegalRepresentative>): void {
		this.legalRepresentative = { ...this.legalRepresentative, ...data };
		this.updatedAt = new Date();
	}

	updateCommercialContact(data: Partial<CommercialContact>): void {
		this.commercialContact = { ...this.commercialContact, ...data };
		this.updatedAt = new Date();
	}

	updateAddress(address: Partial<EnterpriseAddress>): void {
		this.address = { ...this.address, ...address };
		this.updatedAt = new Date();
	}

	updateFinanceContact(contact: Partial<FinanceContact>): void {
		this.financeContact = { ...this.financeContact, ...contact };
		this.updatedAt = new Date();
	}

	updateOperatingCities(cities: string[]): void {
		this.operatingCities = cities;
		this.updatedAt = new Date();
	}

	updateCertifications(certifications: Partial<EnterpriseCertifications>): void {
		this.certifications = { ...this.certifications, ...certifications };
		this.updatedAt = new Date();
	}

	updateIsos(isos: Partial<EnterpriseIsos>): void {
		this.isos = { ...this.isos, ...isos };
		this.updatedAt = new Date();
	}

	updateSeals(seals: Partial<EnterpriseSeals>): void {
		this.seals = { ...this.seals, ...seals };
		this.updatedAt = new Date();
	}

	updateInstitutionalDescription(description: string | null): void {
		this.institutionalDescription = description;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			...this.toBaseDto(),
			companyName: this.companyName,
			companyTradeName: this.companyTradeName,
			cnpj: this.cnpj.getValue(),
			foundationDate: this.foundationDate,
			legalRepresentativeName: this.legalRepresentative.name,
			legalRepresentativeEmail: this.legalRepresentative.email,
			legalRepresentativePhone: this.legalRepresentative.phone,
			commercialEmail: this.commercialContact.email,
			commercialPhone: this.commercialContact.phone,
			street: this.address.street,
			number: this.address.number,
			block: this.address.block,
			district: this.address.district,
			city: this.address.city,
			state: this.address.state,
			zipCode: this.address.zipCode,
			financeContactName: this.financeContact.name,
			financeContactPhone: this.financeContact.phone,
			financeContactEmail: this.financeContact.email,
			operatingCities: this.operatingCities,
			hasCivilLiabilityInsurance: this.certifications.hasCivilLiabilityInsurance,
			hasLegalAdvice: this.certifications.hasLegalAdvice,
			hasAccountingAdvice: this.certifications.hasAccountingAdvice,
			hasWorkSafetyAdvice: this.certifications.hasWorkSafetyAdvice,
			hasTechnicalManager: this.certifications.hasTechnicalManager,
			hasProfessionalCouncilRegularity: this.certifications.hasProfessionalCouncilRegularity,
			hasLgpdCompliance: this.certifications.hasLgpdCompliance,
			hasNr1Compliance: this.certifications.hasNr1Compliance,
			hasComplianceProgram: this.certifications.hasComplianceProgram,
			hasIso9001: this.isos.hasIso9001,
			hasIso14001: this.isos.hasIso14001,
			hasIso45001: this.isos.hasIso45001,
			hasIso37001: this.isos.hasIso37001,
			hasIso19600_37301: this.isos.hasIso19600_37301,
			hasEsg: this.seals.hasEsg,
			hasGreenSeal: this.seals.hasGreenSeal,
			hasChildFriendlySeal: this.seals.hasChildFriendlySeal,
			hasCarbonFreeSeal: this.seals.hasCarbonFreeSeal,
			hasEuRecicloSeal: this.seals.hasEuRecicloSeal,
			hasReclameAquiSeal: this.seals.hasReclameAquiSeal,
			hasCipa: this.seals.hasCipa,
			nrs: this.nrs,
			otherCertifications: this.otherCertifications,
			ownCertifications: this.ownCertifications,
			awards: this.awards,
			institutionalDescription: this.institutionalDescription,
			portfolioUrl: this.portfolioUrl,
		};
	}

	static create(props: EnterpriseProfileProps): EnterpriseProfile {
		return new EnterpriseProfile(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		companyName: string;
		companyTradeName: string;
		cnpj: string;
		foundationDate: Date;
		legalRepresentativeName: string;
		legalRepresentativeEmail: string;
		legalRepresentativePhone: string | null;
		commercialEmail: string;
		commercialPhone: string;
		street: string;
		number: string;
		block: string | null;
		district: string;
		city: string;
		state: string;
		zipCode: string;
		financeContactName: string;
		financeContactPhone: string;
		financeContactEmail: string;
		operatingCities: string[] | null;
		hasCivilLiabilityInsurance: boolean | null;
		hasLegalAdvice: boolean | null;
		hasAccountingAdvice: boolean | null;
		hasWorkSafetyAdvice: boolean | null;
		hasTechnicalManager: boolean | null;
		hasProfessionalCouncilRegularity: boolean | null;
		hasLgpdCompliance: boolean | null;
		hasNr1Compliance: boolean | null;
		hasComplianceProgram: boolean | null;
		hasIso9001: boolean | null;
		hasIso14001: boolean | null;
		hasIso45001: boolean | null;
		hasIso37001: boolean | null;
		hasIso19600_37301: boolean | null;
		hasEsg: boolean | null;
		hasGreenSeal: boolean | null;
		hasChildFriendlySeal: boolean | null;
		hasCarbonFreeSeal: boolean | null;
		hasEuRecicloSeal: boolean | null;
		hasReclameAquiSeal: boolean | null;
		hasCipa: boolean | null;
		nrs: string | null;
		otherCertifications: string | null;
		ownCertifications: string | null;
		awards: string | null;
		institutionalDescription: string | null;
		portfolioUrl: string | null;
		status: ProfileStatus;
		createdAt: Date;
		updatedAt: Date;
		deletedAt?: Date | null;
	}): EnterpriseProfile {
		const { CnpjVO } = require("../value-objects/cnpj.vo");
		const cnpjVO = CnpjVO.create(data.cnpj);

		return new EnterpriseProfile({
			id: data.id,
			userId: data.userId,
			companyName: data.companyName,
			companyTradeName: data.companyTradeName,
			cnpj: cnpjVO,
			foundationDate: data.foundationDate,
			legalRepresentative: {
				name: data.legalRepresentativeName,
				email: data.legalRepresentativeEmail,
				phone: data.legalRepresentativePhone || undefined,
			},
			commercialContact: {
				email: data.commercialEmail,
				phone: data.commercialPhone,
			},
			address: {
				street: data.street,
				number: data.number,
				block: data.block,
				district: data.district,
				city: data.city,
				state: data.state,
				zipCode: data.zipCode,
			},
			financeContact: {
				name: data.financeContactName,
				email: data.financeContactEmail,
				phone: data.financeContactPhone,
			},
			operatingCities: data.operatingCities,
			certifications: {
				hasCivilLiabilityInsurance: data.hasCivilLiabilityInsurance ?? false,
				hasLegalAdvice: data.hasLegalAdvice ?? false,
				hasAccountingAdvice: data.hasAccountingAdvice ?? false,
				hasWorkSafetyAdvice: data.hasWorkSafetyAdvice ?? false,
				hasTechnicalManager: data.hasTechnicalManager ?? false,
				hasProfessionalCouncilRegularity: data.hasProfessionalCouncilRegularity ?? false,
				hasLgpdCompliance: data.hasLgpdCompliance ?? false,
				hasNr1Compliance: data.hasNr1Compliance ?? false,
				hasComplianceProgram: data.hasComplianceProgram ?? false,
			},
			isos: {
				hasIso9001: data.hasIso9001 ?? false,
				hasIso14001: data.hasIso14001 ?? false,
				hasIso45001: data.hasIso45001 ?? false,
				hasIso37001: data.hasIso37001 ?? false,
				hasIso19600_37301: data.hasIso19600_37301 ?? false,
			},
			seals: {
				hasEsg: data.hasEsg ?? false,
				hasGreenSeal: data.hasGreenSeal ?? false,
				hasChildFriendlySeal: data.hasChildFriendlySeal ?? false,
				hasCarbonFreeSeal: data.hasCarbonFreeSeal ?? false,
				hasEuRecicloSeal: data.hasEuRecicloSeal ?? false,
				hasReclameAquiSeal: data.hasReclameAquiSeal ?? false,
				hasCipa: data.hasCipa ?? false,
			},
			nrs: data.nrs,
			otherCertifications: data.otherCertifications,
			ownCertifications: data.ownCertifications,
			awards: data.awards,
			institutionalDescription: data.institutionalDescription,
			portfolioUrl: data.portfolioUrl,
			status: data.status,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
			deletedAt: data.deletedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/marketing.entity.ts =====
import type { ProfileStatus } from "../enums/profile-status";
import type { CnpjVO } from "../value-objects/cnpj.vo";
import { BaseProfile } from "./base-profile.entity";

interface MarketingAddress {
	street: string;
	number: string;
	block: string | null;
	district: string;
	city: string;
	state: string;
	zipCode: string;
}

interface LegalRepresentative {
	name: string;
	email: string;
	phone?: string;
}

interface CommercialContact {
	email: string;
	phone: string;
}

interface FinanceContact {
	name: string;
	email: string;
	phone: string;
}

interface MarketingProfileProps {
	id: string;
	userId: string;
	companyName: string;
	companyTradeName: string;
	cnpj: CnpjVO;
	foundationDate: Date;
	legalRepresentative: LegalRepresentative;
	commercialContact: CommercialContact;
	address: MarketingAddress;
	financeContact: FinanceContact;
	operatingCities: string[] | null;
	status: ProfileStatus;
	createdAt: Date;
	updatedAt: Date;
	deletedAt?: Date | null;
}

export class MarketingProfile extends BaseProfile {
	private companyName: string;
	private companyTradeName: string;
	private cnpj: CnpjVO;
	private foundationDate: Date;
	private legalRepresentative: LegalRepresentative;
	private commercialContact: CommercialContact;
	private address: MarketingAddress;
	private financeContact: FinanceContact;
	private operatingCities: string[] | null;

	constructor(props: MarketingProfileProps) {
		super(
			props.id,
			props.userId,
			props.status,
			props.createdAt,
			props.updatedAt,
			props.deletedAt ?? null,
		);
		this.companyName = props.companyName;
		this.companyTradeName = props.companyTradeName;
		this.cnpj = props.cnpj;
		this.foundationDate = props.foundationDate;
		this.legalRepresentative = props.legalRepresentative;
		this.commercialContact = props.commercialContact;
		this.address = props.address;
		this.financeContact = props.financeContact;
		this.operatingCities = props.operatingCities;
	}

	getCompanyName(): string {
		return this.companyName;
	}

	getCompanyTradeName(): string {
		return this.companyTradeName;
	}

	getCnpj(): string {
		return this.cnpj.getValue();
	}

	getFoundationDate(): Date {
		return this.foundationDate;
	}

	getLegalRepresentative(): LegalRepresentative {
		return { ...this.legalRepresentative };
	}

	getCommercialContact(): CommercialContact {
		return { ...this.commercialContact };
	}

	getAddress(): MarketingAddress {
		return { ...this.address };
	}

	getFinanceContact(): FinanceContact {
		return { ...this.financeContact };
	}

	getOperatingCities(): string[] | null {
		return this.operatingCities ? [...this.operatingCities] : null;
	}

	updateLegalRepresentative(data: Partial<LegalRepresentative>): void {
		this.legalRepresentative = { ...this.legalRepresentative, ...data };
		this.updatedAt = new Date();
	}

	updateCommercialContact(data: Partial<CommercialContact>): void {
		this.commercialContact = { ...this.commercialContact, ...data };
		this.updatedAt = new Date();
	}

	updateAddress(address: Partial<MarketingAddress>): void {
		this.address = { ...this.address, ...address };
		this.updatedAt = new Date();
	}

	updateFinanceContact(contact: Partial<FinanceContact>): void {
		this.financeContact = { ...this.financeContact, ...contact };
		this.updatedAt = new Date();
	}

	updateOperatingCities(cities: string[] | null): void {
		this.operatingCities = cities;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			...this.toBaseDto(),
			companyName: this.companyName,
			companyTradeName: this.companyTradeName,
			cnpj: this.cnpj.getValue(),
			foundationDate: this.foundationDate,
			legalRepresentativeName: this.legalRepresentative.name,
			legalRepresentativeEmail: this.legalRepresentative.email,
			legalRepresentativePhone: this.legalRepresentative.phone,
			commercialEmail: this.commercialContact.email,
			commercialPhone: this.commercialContact.phone,
			street: this.address.street,
			number: this.address.number,
			block: this.address.block,
			district: this.address.district,
			city: this.address.city,
			state: this.address.state,
			zipCode: this.address.zipCode,
			financeContactName: this.financeContact.name,
			financeContactPhone: this.financeContact.phone,
			financeContactEmail: this.financeContact.email,
			operatingCities: this.operatingCities,
		};
	}

	static create(props: MarketingProfileProps): MarketingProfile {
		return new MarketingProfile(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		companyName: string;
		companyTradeName: string;
		cnpj: string;
		foundationDate: Date;
		legalRepresentativeName: string;
		legalRepresentativeEmail: string;
		legalRepresentativePhone: string | null;
		commercialEmail: string;
		commercialPhone: string;
		street: string;
		number: string;
		block: string | null;
		district: string;
		city: string;
		state: string;
		zipCode: string;
		financeContactName: string;
		financeContactPhone: string;
		financeContactEmail: string;
		operatingCities: string[] | null;
		status: ProfileStatus;
		createdAt: Date;
		updatedAt: Date;
		deletedAt?: Date | null;
	}): MarketingProfile {
		const { CnpjVO } = require("../value-objects/cnpj.vo");
		const cnpjVO = CnpjVO.create(data.cnpj);

		return new MarketingProfile({
			id: data.id,
			userId: data.userId,
			companyName: data.companyName,
			companyTradeName: data.companyTradeName,
			cnpj: cnpjVO,
			foundationDate: data.foundationDate,
			legalRepresentative: {
				name: data.legalRepresentativeName,
				email: data.legalRepresentativeEmail,
				phone: data.legalRepresentativePhone || undefined,
			},
			commercialContact: {
				email: data.commercialEmail,
				phone: data.commercialPhone,
			},
			address: {
				street: data.street,
				number: data.number,
				block: data.block,
				district: data.district,
				city: data.city,
				state: data.state,
				zipCode: data.zipCode,
			},
			financeContact: {
				name: data.financeContactName,
				email: data.financeContactEmail,
				phone: data.financeContactPhone,
			},
			operatingCities: data.operatingCities,
			status: data.status,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
			deletedAt: data.deletedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/profile/domain/entities/local-company.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { ProfileStatus } from "../enums/profile-status";
import type { CnpjVO } from "../value-objects/cnpj.vo";
import type { CpfVO } from "../value-objects/cpf.vo";
import { BaseProfile } from "./base-profile.entity";

interface LocalCompanyAddress {
	street: string;
	number: string;
	block: string | null;
	district: string;
	city: string;
	state: string;
	zipCode: string;
}

interface LegalRepresentative {
	name: string;
	email: string;
	phone?: string;
}

interface CommercialContact {
	email: string;
	phone: string;
}

interface FinanceContact {
	name: string;
	email: string;
	phone: string;
}

interface LocalCompanyProfileProps {
	id: string;
	userId: string;
	companyName: string;
	companyTradeName: string;
	document: CpfVO | CnpjVO;
	foundationDate: Date;
	legalRepresentative: LegalRepresentative;
	commercialContact: CommercialContact;
	address: LocalCompanyAddress;
	financeContact: FinanceContact;
	photos: string[] | null;
	status: ProfileStatus;
	createdAt: Date;
	updatedAt: Date;
	deletedAt?: Date | null;
}

export class LocalCompanyProfile extends BaseProfile {
	private companyName: string;
	private companyTradeName: string;
	private document: CpfVO | CnpjVO;
	private foundationDate: Date;
	private legalRepresentative: LegalRepresentative;
	private commercialContact: CommercialContact;
	private address: LocalCompanyAddress;
	private financeContact: FinanceContact;
	private photos: string[] | null;

	constructor(props: LocalCompanyProfileProps) {
		super(
			props.id,
			props.userId,
			props.status,
			props.createdAt,
			props.updatedAt,
			props.deletedAt ?? null,
		);
		this.companyName = props.companyName;
		this.companyTradeName = props.companyTradeName;
		this.document = props.document;
		this.foundationDate = props.foundationDate;
		this.legalRepresentative = props.legalRepresentative;
		this.commercialContact = props.commercialContact;
		this.address = props.address;
		this.financeContact = props.financeContact;
		this.photos = props.photos;
	}

	getCompanyName(): string {
		return this.companyName;
	}

	getCompanyTradeName(): string {
		return this.companyTradeName;
	}

	getDocument(): string {
		return this.document.getValue();
	}

	getFoundationDate(): Date {
		return this.foundationDate;
	}

	getLegalRepresentative(): LegalRepresentative {
		return { ...this.legalRepresentative };
	}

	getCommercialContact(): CommercialContact {
		return { ...this.commercialContact };
	}

	getAddress(): LocalCompanyAddress {
		return { ...this.address };
	}

	getFinanceContact(): FinanceContact {
		return { ...this.financeContact };
	}

	getPhotos(): string[] | null {
		return this.photos ? [...this.photos] : null;
	}

	isCnpj(): boolean {
		return this.document.getValue().length === 14;
	}

	isCpf(): boolean {
		return this.document.getValue().length === 11;
	}

	updateLegalRepresentative(data: Partial<LegalRepresentative>): void {
		this.legalRepresentative = { ...this.legalRepresentative, ...data };
		this.updatedAt = new Date();
	}

	updateCommercialContact(data: Partial<CommercialContact>): void {
		this.commercialContact = { ...this.commercialContact, ...data };
		this.updatedAt = new Date();
	}

	updateAddress(address: Partial<LocalCompanyAddress>): void {
		this.address = { ...this.address, ...address };
		this.updatedAt = new Date();
	}

	updateFinanceContact(contact: Partial<FinanceContact>): void {
		this.financeContact = { ...this.financeContact, ...contact };
		this.updatedAt = new Date();
	}

	updatePhotos(photos: string[]): void {
		if (photos.length > 3) {
			throw new BusinessError("Máximo de 3 fotos permitido.", "MAX_PHOTOS_EXCEEDED");
		}
		this.photos = photos;
		this.updatedAt = new Date();
	}

	addPhoto(photoUrl: string): void {
		if (this.photos && this.photos.length >= 3) {
			throw new BusinessError("Máximo de 3 fotos permitido.", "MAX_PHOTOS_EXCEEDED");
		}
		this.photos = this.photos ? [...this.photos, photoUrl] : [photoUrl];
		this.updatedAt = new Date();
	}

	removePhoto(photoUrl: string): void {
		if (this.photos) {
			this.photos = this.photos.filter((p) => p !== photoUrl);
			this.updatedAt = new Date();
		}
	}

	toDto() {
		return {
			...this.toBaseDto(),
			companyName: this.companyName,
			companyTradeName: this.companyTradeName,
			document: this.document.getValue(),
			foundationDate: this.foundationDate,
			legalRepresentativeName: this.legalRepresentative.name,
			legalRepresentativeEmail: this.legalRepresentative.email,
			legalRepresentativePhone: this.legalRepresentative.phone,
			commercialEmail: this.commercialContact.email,
			commercialPhone: this.commercialContact.phone,
			street: this.address.street,
			number: this.address.number,
			block: this.address.block,
			district: this.address.district,
			city: this.address.city,
			state: this.address.state,
			zipCode: this.address.zipCode,
			financeContactName: this.financeContact.name,
			financeContactPhone: this.financeContact.phone,
			financeContactEmail: this.financeContact.email,
			photos: this.photos,
		};
	}

	static create(props: LocalCompanyProfileProps): LocalCompanyProfile {
		return new LocalCompanyProfile(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		companyName: string;
		companyTradeName: string;
		document: string;
		foundationDate: Date;
		legalRepresentativeName: string;
		legalRepresentativeEmail: string;
		legalRepresentativePhone: string | null;
		commercialEmail: string;
		commercialPhone: string;
		street: string;
		number: string;
		block: string | null;
		district: string;
		city: string;
		state: string;
		zipCode: string;
		financeContactName: string;
		financeContactPhone: string;
		financeContactEmail: string;
		photos: string[] | null;
		status: ProfileStatus;
		createdAt: Date;
		updatedAt: Date;
		deletedAt?: Date | null;
	}): LocalCompanyProfile {
		const { CpfVO } = require("../value-objects/cpf.vo");
		const { CnpjVO } = require("../value-objects/cnpj.vo");

		// Determina se é CPF ou CNPJ pelo comprimento
		const cleanDocument = data.document.replace(/\D/g, "");
		const documentVO = cleanDocument.length === 11
			? CpfVO.create(data.document)
			: CnpjVO.create(data.document);

		return new LocalCompanyProfile({
			id: data.id,
			userId: data.userId,
			companyName: data.companyName,
			companyTradeName: data.companyTradeName,
			document: documentVO,
			foundationDate: data.foundationDate,
			legalRepresentative: {
				name: data.legalRepresentativeName,
				email: data.legalRepresentativeEmail,
				phone: data.legalRepresentativePhone || undefined,
			},
			commercialContact: {
				email: data.commercialEmail,
				phone: data.commercialPhone,
			},
			address: {
				street: data.street,
				number: data.number,
				block: data.block,
				district: data.district,
				city: data.city,
				state: data.state,
				zipCode: data.zipCode,
			},
			financeContact: {
				name: data.financeContactName,
				email: data.financeContactEmail,
				phone: data.financeContactPhone,
			},
			photos: data.photos,
			status: data.status,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
			deletedAt: data.deletedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/profile/domain/repositories/enterprise.repository.ts =====
import type { EnterpriseProfile } from "../entities/enterprise.entity";

export interface IEnterpriseProfileRepository {
	findById(id: string): Promise<EnterpriseProfile | null>;
	findByUserId(userId: string): Promise<EnterpriseProfile | null>;
	findByCnpj(cnpj: string): Promise<EnterpriseProfile | null>;

	save(profile: EnterpriseProfile): Promise<void>;
	softDelete(id: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/profile/domain/repositories/local-company.repository.ts =====
import type { LocalCompanyProfile } from "../entities/local-company.entity";

export interface ILocalCompanyProfileRepository {
	findById(id: string): Promise<LocalCompanyProfile | null>;
	findByUserId(userId: string): Promise<LocalCompanyProfile | null>;
	findByDocument(document: string): Promise<LocalCompanyProfile | null>;

	save(profile: LocalCompanyProfile): Promise<void>;
	softDelete(id: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/profile/domain/repositories/marketing.repository.ts =====
import type { MarketingProfile } from "../entities/marketing.entity";

export interface IMarketingProfileRepository {
	findById(id: string): Promise<MarketingProfile | null>;
	findByUserId(userId: string): Promise<MarketingProfile | null>;
	findByCnpj(cnpj: string): Promise<MarketingProfile | null>;

	save(profile: MarketingProfile): Promise<void>;
	softDelete(id: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/profile/domain/repositories/resident.repository.ts =====
import type { ResidentProfile } from "../entities/resident.entity";

export interface IResidentProfileRepository {
	findById(id: string): Promise<ResidentProfile | null>;
	findByUserId(userId: string): Promise<ResidentProfile | null>;
	findByCpf(cpf: string): Promise<ResidentProfile | null>;

	save(profile: ResidentProfile): Promise<void>;
	softDelete(id: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/profile/domain/repositories/syndic.repository.ts =====
import type { SyndicProfile } from "../entities/syndic.entity";

export interface ISyndicProfileRepository {
	findById(id: string): Promise<SyndicProfile | null>;
	findByUserId(userId: string): Promise<SyndicProfile | null>;
	findByDocument(document: string): Promise<SyndicProfile | null>;

	save(profile: SyndicProfile): Promise<void>;
	softDelete(id: string): Promise<void>;
	restore(id: string): Promise<void>;
	hardDelete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/profile/domain/enums/profile-status.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { profileStatus } from "../../../../infrastructure/database/drizzle/schema/enums";

export const profileStatusSchema = createSelectSchema(profileStatus);

export type ProfileStatus = z.infer<typeof profileStatusSchema>;

// Export enum for easier usage
export { ProfileStatus as ProfileStatusEnum } from "./profile-status.enum";
\n\n===== FILE: ./src/modules/profile/domain/enums/syndic-type.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { syndicType } from "../../../../infrastructure/database/drizzle/schema/enums";

export const syndicTypeSchema = createSelectSchema(syndicType);

export type SyndicType = z.infer<typeof syndicTypeSchema>;
\n\n===== FILE: ./src/modules/profile/domain/enums/index.ts =====
export * from "./profile-status";
export * from "./syndic-type";
\n\n===== FILE: ./src/modules/profile/domain/enums/profile-status.enum.ts =====
/**
 * Status do Perfil
 * 
 * Responsabilidade: Gerenciar a completude e disponibilidade do perfil
 * NÃO confundir com status de assinatura/plano (módulo subscription)
 */
export enum ProfileStatus {
	/**
	 * Perfil em criação
	 * Campos obrigatórios ainda não foram todos preenchidos
	 */
	INCOMPLETE = "incomplete",

	/**
	 * Perfil completo
	 * Todos os campos obrigatórios foram preenchidos
	 */
	COMPLETE = "complete",

	/**
	 * Perfil inativo
	 * Usuário desativou o perfil ou foi banido/suspenso pelo admin
	 */
	INACTIVE = "inactive",
}
\n\n===== FILE: ./src/modules/profile/domain/value-objects/cpf.vo.ts =====
import { BusinessError } from "../../../../core/errors";

export class CpfVO {
	private constructor(private readonly value: string) {}

	static create(value: string): CpfVO {
		const cleaned = value.replace(/\D/g, "");

		if (cleaned.length !== 11) {
			throw new BusinessError("CPF deve ter 11 dígitos", "INVALID_CPF");
		}

		if (!CpfVO.isValid(cleaned)) {
			throw new BusinessError("CPF inválido", "INVALID_CPF");
		}

		return new CpfVO(cleaned);
	}

	private static isValid(cpf: string): boolean {
		if (/^(\d)\1{10}$/.test(cpf)) return false;

		const digits = cpf.split("").map(Number);

		let sum = 0;
		for (let i = 0; i < 9; i++) {
			sum += (digits[i] ?? 0) * (10 - i);
		}
		let digit = 11 - (sum % 11);
		if (digit >= 10) digit = 0;
		if (digit !== (digits[9] ?? -1)) return false;

		sum = 0;
		for (let i = 0; i < 10; i++) {
			sum += (digits[i] ?? 0) * (11 - i);
		}
		digit = 11 - (sum % 11);
		if (digit >= 10) digit = 0;
		if (digit !== (digits[10] ?? -1)) return false;

		return true;
	}

	getValue(): string {
		return this.value;
	}

	getFormatted(): string {
		return this.value.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4");
	}

	equals(other: CpfVO): boolean {
		return this.value === other.value;
	}
}
\n\n===== FILE: ./src/modules/profile/domain/value-objects/cnpj.vo.ts =====
import { BusinessError } from "../../../../core/errors";

export class CnpjVO {
	private constructor(private readonly value: string) {}

	static create(value: string): CnpjVO {
		const cleaned = value.replace(/\D/g, "");

		if (cleaned.length !== 14) {
			throw new BusinessError("CNPJ deve ter 14 dígitos", "INVALID_CNPJ");
		}

		if (!CnpjVO.isValid(cleaned)) {
			throw new BusinessError("CNPJ inválido", "INVALID_CNPJ");
		}

		return new CnpjVO(cleaned);
	}

	private static isValid(cnpj: string): boolean {
		if (/^(\d)\1{13}$/.test(cnpj)) return false;

		const digits = cnpj.split("").map(Number);
		const weights1 = [5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2];
		const weights2 = [6, 5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2];

		let sum = 0;
		for (let i = 0; i < 12; i++) {
			sum += (digits[i] ?? 0) * (weights1[i] ?? 0);
		}
		let digit = sum % 11;
		digit = digit < 2 ? 0 : 11 - digit;
		if (digit !== (digits[12] ?? -1)) return false;

		sum = 0;
		for (let i = 0; i < 13; i++) {
			sum += (digits[i] ?? 0) * (weights2[i] ?? 0);
		}
		digit = sum % 11;
		digit = digit < 2 ? 0 : 11 - digit;
		if (digit !== (digits[13] ?? -1)) return false;

		return true;
	}

	getValue(): string {
		return this.value;
	}

	getFormatted(): string {
		return this.value.replace(
			/(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})/,
			"$1.$2.$3/$4-$5",
		);
	}

	equals(other: CnpjVO): boolean {
		return this.value === other.value;
	}
}
\n\n===== FILE: ./src/modules/profile/domain/value-objects/index.ts =====
export * from "./cpf.vo";
export * from "./cnpj.vo";
\n\n===== FILE: ./src/modules/profile/domain/services/profile.service.ts =====
import type { UseCaseDependencies } from "../../../../core/contracts/base-use-case";
import type { User } from "../../../user/domain/entities/user.entity";
import type { IEnterpriseProfileRepository } from "../repositories/enterprise.repository";
import type { ILocalCompanyProfileRepository } from "../repositories/local-company.repository";
import type { IMarketingProfileRepository } from "../repositories/marketing.repository";
import type { IResidentProfileRepository } from "../repositories/resident.repository";
import type { ISyndicProfileRepository } from "../repositories/syndic.repository";




interface ProfileServiceDependencies extends UseCaseDependencies {
    residentProfileRepo: IResidentProfileRepository,
    syndicProfileRepo: ISyndicProfileRepository,
    enterpriseProfileRepo: IEnterpriseProfileRepository,
    localCompanyProfileRepo: ILocalCompanyProfileRepository,
    marketingProfileRepo: IMarketingProfileRepository
}

export class ProfileService {
    constructor(private readonly deps: ProfileServiceDependencies) {}


    /**
     * Cria ou atualiza um perfil baseado na Role do usuário.
     * Se o usuário já tiver um perfil daquele role, retorna ele (para atualização).
     */
    async getOrCreateProfile(user: User) {}
}\n\n===== FILE: ./src/modules/profile/application/use-cases/create-resident-profile.use-case.ts =====
import type {
	CreateResidentProfile,
	ResidentProfileOutput,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { ResidentProfile } from "../../domain/entities/resident.entity";
import type { IResidentProfileRepository } from "../../domain/repositories/resident.repository";
import { CpfVO } from "../../domain/value-objects/cpf.vo";

interface CreateResidentProfileUseCaseDependencies extends UseCaseDependencies {
	residentProfileRepository: IResidentProfileRepository;
	userRepository: IUserRepository;
}

export class CreateResidentProfileUseCase extends TransactionalUseCase<
	CreateResidentProfile,
	ResidentProfileOutput
> {
	constructor(private readonly deps: CreateResidentProfileUseCaseDependencies) {
		super(deps);
	}

	async execute(input: CreateResidentProfile): Promise<ResidentProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando criação de perfil de morador",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Validações de negócio
			await this.validateUserExists(input.userId);
			await this.validateProfileDoesNotExist(input.userId);
			await this.validateCpfIsUnique(input.cpf);

			// 2. Criar entidade de perfil
			const profile = this.createProfile(input);

			// 3. Persistir
			await this.deps.residentProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de morador criado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateUserExists(userId: string): Promise<void> {
		const user = await this.deps.userRepository.findById(userId);
		if (!user) {
			throw new NotFoundError("Usuário não encontrado");
		}
	}

	private async validateProfileDoesNotExist(userId: string): Promise<void> {
		const existingProfile =
			await this.deps.residentProfileRepository.findByUserId(userId);
		if (existingProfile) {
			throw new BusinessError(
				"Este usuário já possui um perfil de morador",
				"PROFILE_ALREADY_EXISTS",
			);
		}
	}

	private async validateCpfIsUnique(cpf: string): Promise<void> {
		const existingProfile =
			await this.deps.residentProfileRepository.findByCpf(cpf);
		if (existingProfile) {
			throw new BusinessError("Este CPF já está cadastrado", "CPF_ALREADY_EXISTS");
		}
	}

	// --- Criação ---

	private createProfile(input: CreateResidentProfile): ResidentProfile {
		// Criar o VO do CPF (já validado pelo schema)
		const cpfVO = CpfVO.create(input.cpf);

		return ResidentProfile.create({
			id: generateId(),
			userId: input.userId,
			birthDate: input.birthDate,
			cpf: cpfVO,
			address: {
				street: input.address.street,
				number: input.address.number,
				block: input.address.block,
				district: input.address.district,
				city: input.address.city,
				state: input.address.state,
				zipCode: input.address.zipCode,
			},
			buildingNameOrCode: input.buildingNameOrCode,
			status: "incomplete", // Sempre começa como incomplete
			createdAt: new Date(),
			updatedAt: new Date(),
		});
	}

	// --- Response ---

	private buildOutput(profile: ResidentProfile): ResidentProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/create-local-company-profile.use-case.ts =====
import type {
	CreateLocalCompanyProfile,
	LocalCompanyProfileOutput,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { LocalCompanyProfile } from "../../domain/entities/local-company.entity";
import type { ILocalCompanyProfileRepository } from "../../domain/repositories/local-company.repository";
import { CnpjVO } from "../../domain/value-objects/cnpj.vo";
import { CpfVO } from "../../domain/value-objects/cpf.vo";

interface CreateLocalCompanyProfileUseCaseDependencies
	extends UseCaseDependencies {
	localCompanyProfileRepository: ILocalCompanyProfileRepository;
	userRepository: IUserRepository;
}

export class CreateLocalCompanyProfileUseCase extends TransactionalUseCase<
	CreateLocalCompanyProfile,
	LocalCompanyProfileOutput
> {
	constructor(
		private readonly deps: CreateLocalCompanyProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: CreateLocalCompanyProfile,
	): Promise<LocalCompanyProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando criação de perfil de comércio local",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Validações de negócio
			await this.validateUserExists(input.userId);
			await this.validateProfileDoesNotExist(input.userId);
			await this.validateDocumentIsUnique(input.document);

			// 2. Criar entidade de perfil
			const profile = this.createProfile(input);

			// 3. Persistir
			await this.deps.localCompanyProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de comércio local criado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateUserExists(userId: string): Promise<void> {
		const user = await this.deps.userRepository.findById(userId);
		if (!user) {
			throw new NotFoundError("Usuário não encontrado");
		}
	}

	private async validateProfileDoesNotExist(userId: string): Promise<void> {
		const existingProfile =
			await this.deps.localCompanyProfileRepository.findByUserId(userId);
		if (existingProfile) {
			throw new BusinessError(
				"Este usuário já possui um perfil de comércio local",
				"PROFILE_ALREADY_EXISTS",
			);
		}
	}

	private async validateDocumentIsUnique(document: string): Promise<void> {
		const existingProfile =
			await this.deps.localCompanyProfileRepository.findByDocument(document);
		if (existingProfile) {
			throw new BusinessError(
				"Este documento já está cadastrado",
				"DOCUMENT_ALREADY_EXISTS",
			);
		}
	}

	// --- Criação ---

	private createProfile(
		input: CreateLocalCompanyProfile,
	): LocalCompanyProfile {
		// Determina se é CPF ou CNPJ e cria o VO apropriado
		const cleanDocument = input.document.replace(/\D/g, "");
		const documentVO =
			cleanDocument.length === 11
				? CpfVO.create(input.document)
				: CnpjVO.create(input.document);

		return LocalCompanyProfile.create({
			id: generateId(),
			userId: input.userId,
			companyName: input.companyName,
			companyTradeName: input.companyTradeName,
			document: documentVO,
			foundationDate: input.foundationDate,
			legalRepresentativeName: input.legalRepresentativeName,
			legalRepresentativeEmail: input.legalRepresentativeEmail,
			commercialEmail: input.commercialEmail,
			commercialPhone: input.commercialPhone,
			address: input.address,
			financeContact: input.financeContact,
			photos: input.photos ?? null,
			status: "incomplete",
			createdAt: new Date(),
			updatedAt: new Date(),
		});
	}

	// --- Response ---

	private buildOutput(profile: LocalCompanyProfile): LocalCompanyProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/create-syndic-profile.use-case.ts =====
import type {
	CreateSyndicProfile,
	SyndicProfileOutput,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { SyndicProfile } from "../../domain/entities/syndic.entity";
import type { ISyndicProfileRepository } from "../../domain/repositories/syndic.repository";
import { CnpjVO } from "../../domain/value-objects/cnpj.vo";
import { CpfVO } from "../../domain/value-objects/cpf.vo";

interface CreateSyndicProfileUseCaseDependencies extends UseCaseDependencies {
	syndicProfileRepository: ISyndicProfileRepository;
	userRepository: IUserRepository;
}

export class CreateSyndicProfileUseCase extends TransactionalUseCase<
	CreateSyndicProfile,
	SyndicProfileOutput
> {
	constructor(
		private readonly deps: CreateSyndicProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(input: CreateSyndicProfile): Promise<SyndicProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando criação de perfil de síndico",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Validações de negócio
			await this.validateUserExists(input.userId);
			await this.validateProfileDoesNotExist(input.userId);
			await this.validateDocumentIsUnique(input.document);
			this.validateExperience(input.experienceYears, input.buildingsCount);

			// 2. Criar entidade de perfil
			const profile = this.createProfile(input);

			// 3. Persistir
			await this.deps.syndicProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de síndico criado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateUserExists(userId: string): Promise<void> {
		const user = await this.deps.userRepository.findById(userId);
		if (!user) {
			throw new NotFoundError("Usuário não encontrado");
		}
	}

	private async validateProfileDoesNotExist(userId: string): Promise<void> {
		const existingProfile =
			await this.deps.syndicProfileRepository.findByUserId(userId);
		if (existingProfile) {
			throw new BusinessError(
				"Este usuário já possui um perfil de síndico",
				"PROFILE_ALREADY_EXISTS",
			);
		}
	}

	private async validateDocumentIsUnique(document: string): Promise<void> {
		const existingProfile =
			await this.deps.syndicProfileRepository.findByDocument(document);
		if (existingProfile) {
			throw new BusinessError(
				"Este documento já está cadastrado",
				"DOCUMENT_ALREADY_EXISTS",
			);
		}
	}

	private validateExperience(
		experienceYears: number,
		buildingsCount: number,
	): void {
		if (experienceYears < 0) {
			throw new BusinessError(
				"Anos de experiência não pode ser negativo",
				"INVALID_EXPERIENCE_YEARS",
			);
		}

		if (buildingsCount < 0) {
			throw new BusinessError(
				"Quantidade de prédios não pode ser negativa",
				"INVALID_BUILDINGS_COUNT",
			);
		}
	}

	// --- Criação ---

	private createProfile(input: CreateSyndicProfile): SyndicProfile {
		// Determina se é CPF ou CNPJ e cria o VO apropriado
		const cleanDocument = input.document.replace(/\D/g, "");
		const documentVO =
			cleanDocument.length === 11
				? CpfVO.create(input.document)
				: CnpjVO.create(input.document);

		return SyndicProfile.create({
			id: generateId(),
			userId: input.userId,
			birthDate: input.birthDate,
			document: documentVO,
			address: input.address,
			type: input.type,
			experienceYears: input.experienceYears,
			buildingsCount: input.buildingsCount,
			operatingCities: input.operatingCities.length > 0 ? input.operatingCities : null,
			certifications: {
				hasAbracsMember: input.hasAbracsMember,
				hasCivilLiabilityInsurance: input.hasCivilLiabilityInsurance,
				hasLegalAdvice: input.hasLegalAdvice,
				hasAccountingAdvice: input.hasAccountingAdvice,
				hasWorkSafetyAdvice: input.hasWorkSafetyAdvice,
				hasLgpdCompliance: input.hasLgpdCompliance,
				hasNr1Compliance: input.hasNr1Compliance,
				hasComplianceProgram: input.hasComplianceProgram,
				hasReclameAquiSeal: input.hasReclameAquiSeal,
				otherCertifications: input.otherCertifications,
				awards: input.awards,
			},
			miniBio: input.miniBio,
			educationAndCertifications: input.educationAndCertifications,
			linkedAdministrators: input.linkedAdministrators,
			status: "incomplete",
			createdAt: new Date(),
			updatedAt: new Date(),
		});
	}

	// --- Response ---

	private buildOutput(profile: SyndicProfile): SyndicProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/create-marketing-profile.use-case.ts =====
import type {
	CreateMarketingProfile,
	MarketingProfileOutput,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { MarketingProfile } from "../../domain/entities/marketing.entity";
import type { IMarketingProfileRepository } from "../../domain/repositories/marketing.repository";
import { CnpjVO } from "../../domain/value-objects/cnpj.vo";

interface CreateMarketingProfileUseCaseDependencies
	extends UseCaseDependencies {
	marketingProfileRepository: IMarketingProfileRepository;
	userRepository: IUserRepository;
}

export class CreateMarketingProfileUseCase extends TransactionalUseCase<
	CreateMarketingProfile,
	MarketingProfileOutput
> {
	constructor(
		private readonly deps: CreateMarketingProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: CreateMarketingProfile,
	): Promise<MarketingProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando criação de perfil de marketing",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Validações de negócio
			await this.validateUserExists(input.userId);
			await this.validateProfileDoesNotExist(input.userId);
			await this.validateCnpjIsUnique(input.cnpj);

			// 2. Criar entidade de perfil
			const profile = this.createProfile(input);

			// 3. Persistir
			await this.deps.marketingProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de marketing criado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateUserExists(userId: string): Promise<void> {
		const user = await this.deps.userRepository.findById(userId);
		if (!user) {
			throw new NotFoundError("Usuário não encontrado");
		}
	}

	private async validateProfileDoesNotExist(userId: string): Promise<void> {
		const existingProfile =
			await this.deps.marketingProfileRepository.findByUserId(userId);
		if (existingProfile) {
			throw new BusinessError(
				"Este usuário já possui um perfil de marketing",
				"PROFILE_ALREADY_EXISTS",
			);
		}
	}

	private async validateCnpjIsUnique(cnpj: string): Promise<void> {
		const existingProfile =
			await this.deps.marketingProfileRepository.findByCnpj(cnpj);
		if (existingProfile) {
			throw new BusinessError(
				"Este CNPJ já está cadastrado",
				"CNPJ_ALREADY_EXISTS",
			);
		}
	}

	// --- Criação ---

	private createProfile(input: CreateMarketingProfile): MarketingProfile {
		// Criar o VO do CNPJ (já validado pelo schema)
		const cnpjVO = CnpjVO.create(input.cnpj);

		return MarketingProfile.create({
			id: generateId(),
			userId: input.userId,
			logoUrl: input.logoUrl ?? null,
			companyName: input.companyName,
			companyTradeName: input.companyTradeName,
			cnpj: cnpjVO,
			foundationDate: input.foundationDate,
			legalRepresentativeName: input.legalRepresentativeName,
			legalRepresentativeEmail: input.legalRepresentativeEmail,
			commercialEmail: input.commercialEmail,
			commercialPhone: input.commercialPhone,
			address: input.address,
			financeContact: input.financeContact,
			operatingCities: input.operatingCities.length > 0 ? input.operatingCities : null,
			status: "incomplete",
			createdAt: new Date(),
			updatedAt: new Date(),
		});
	}

	// --- Response ---

	private buildOutput(profile: MarketingProfile): MarketingProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/update-marketing-profile.use-case.ts =====
import type {
	MarketingProfileOutput,
	UpdateMarketingProfile,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import type { MarketingProfile } from "../../domain/entities/marketing.entity";
import type { IMarketingProfileRepository } from "../../domain/repositories/marketing.repository";

interface UpdateMarketingProfileUseCaseDependencies
	extends UseCaseDependencies {
	marketingProfileRepository: IMarketingProfileRepository;
}

export class UpdateMarketingProfileUseCase extends TransactionalUseCase<
	UpdateMarketingProfile,
	MarketingProfileOutput
> {
	constructor(
		private readonly deps: UpdateMarketingProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: UpdateMarketingProfile,
	): Promise<MarketingProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando atualização de perfil de marketing",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Buscar perfil existente
			const profile = await this.deps.marketingProfileRepository.findByUserId(
				input.userId,
			);
			if (!profile) {
				throw new NotFoundError("Perfil de marketing não encontrado");
			}

			// 2. Validar CNPJ se estiver sendo alterado
			if (input.cnpj && input.cnpj !== profile.getCnpj()) {
				await this.validateCnpjIsUnique(input.cnpj);
			}

			// 3. Atualizar campos
			if (input.logoUrl !== undefined) {
				profile.updateLogoUrl(input.logoUrl);
			}

			if (input.commercialEmail || input.commercialPhone) {
				profile.updateCommercialContact(
					input.commercialEmail ?? profile.getCommercialEmail(),
					input.commercialPhone ?? profile.getCommercialPhone(),
				);
			}

			if (input.address) {
				profile.updateAddress(input.address);
			}

			if (input.financeContact) {
				profile.updateFinanceContact(input.financeContact);
			}

			if (input.operatingCities !== undefined) {
				profile.updateOperatingCities(
					input.operatingCities.length > 0 ? input.operatingCities : null,
				);
			}

			// 4. Persistir
			await this.deps.marketingProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de marketing atualizado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateCnpjIsUnique(cnpj: string): Promise<void> {
		const existingProfile =
			await this.deps.marketingProfileRepository.findByCnpj(cnpj);
		if (existingProfile) {
			throw new BusinessError(
				"Este CNPJ já está cadastrado",
				"CNPJ_ALREADY_EXISTS",
			);
		}
	}

	// --- Response ---

	private buildOutput(profile: MarketingProfile): MarketingProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/update-local-company-profile.use-case.ts =====
import type {
	LocalCompanyProfileOutput,
	UpdateLocalCompanyProfile,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import type { LocalCompanyProfile } from "../../domain/entities/local-company.entity";
import type { ILocalCompanyProfileRepository } from "../../domain/repositories/local-company.repository";

interface UpdateLocalCompanyProfileUseCaseDependencies
	extends UseCaseDependencies {
	localCompanyProfileRepository: ILocalCompanyProfileRepository;
}

export class UpdateLocalCompanyProfileUseCase extends TransactionalUseCase<
	UpdateLocalCompanyProfile,
	LocalCompanyProfileOutput
> {
	constructor(
		private readonly deps: UpdateLocalCompanyProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: UpdateLocalCompanyProfile,
	): Promise<LocalCompanyProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando atualização de perfil de comércio local",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Buscar perfil existente
			const profile =
				await this.deps.localCompanyProfileRepository.findByUserId(
					input.userId,
				);
			if (!profile) {
				throw new NotFoundError("Perfil de comércio local não encontrado");
			}

			// 2. Validar documento se estiver sendo alterado
			if (input.document && input.document !== profile.getDocument()) {
				await this.validateDocumentIsUnique(input.document);
			}

			// 3. Atualizar campos
			if (input.address) {
				profile.updateAddress(input.address);
			}

			if (input.financeContact) {
				profile.updateFinanceContact(input.financeContact);
			}

			if (input.photos !== undefined) {
				profile.updatePhotos(input.photos ?? []);
			}

			// 4. Persistir
			await this.deps.localCompanyProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de comércio local atualizado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateDocumentIsUnique(document: string): Promise<void> {
		const existingProfile =
			await this.deps.localCompanyProfileRepository.findByDocument(document);
		if (existingProfile) {
			throw new BusinessError(
				"Este documento já está cadastrado",
				"DOCUMENT_ALREADY_EXISTS",
			);
		}
	}

	// --- Response ---

	private buildOutput(profile: LocalCompanyProfile): LocalCompanyProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/update-syndic-profile.use-case.ts =====
import type {
	SyndicProfileOutput,
	UpdateSyndicProfile,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import type { SyndicProfile } from "../../domain/entities/syndic.entity";
import type { ISyndicProfileRepository } from "../../domain/repositories/syndic.repository";

interface UpdateSyndicProfileUseCaseDependencies extends UseCaseDependencies {
	syndicProfileRepository: ISyndicProfileRepository;
}

export class UpdateSyndicProfileUseCase extends TransactionalUseCase<
	UpdateSyndicProfile,
	SyndicProfileOutput
> {
	constructor(private readonly deps: UpdateSyndicProfileUseCaseDependencies) {
		super(deps);
	}

	async execute(input: UpdateSyndicProfile): Promise<SyndicProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando atualização de perfil de síndico",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Buscar perfil existente
			const profile = await this.deps.syndicProfileRepository.findByUserId(
				input.userId,
			);
			if (!profile) {
				throw new NotFoundError("Perfil de síndico não encontrado");
			}

			// 2. Validar documento se estiver sendo alterado
			if (input.document && input.document !== profile.getDocument()) {
				await this.validateDocumentIsUnique(input.document);
			}

			// 3. Atualizar campos
			if (input.address) {
				profile.updateAddress(input.address);
			}

			if (
				input.experienceYears !== undefined ||
				input.buildingsCount !== undefined
			) {
				profile.updateExperience(
					input.experienceYears ?? profile.getExperienceYears(),
					input.buildingsCount ?? profile.getBuildingsCount(),
				);
			}

			if (input.operatingCities !== undefined) {
				profile.updateOperatingCities(input.operatingCities);
			}

			// Atualizar certifications - agrupa os campos booleanos
			const hasCertificationUpdates =
				input.hasAbracsMember !== undefined ||
				input.hasCivilLiabilityInsurance !== undefined ||
				input.hasLegalAdvice !== undefined ||
				input.hasAccountingAdvice !== undefined ||
				input.hasWorkSafetyAdvice !== undefined ||
				input.hasLgpdCompliance !== undefined ||
				input.hasNr1Compliance !== undefined ||
				input.hasComplianceProgram !== undefined ||
				input.hasReclameAquiSeal !== undefined ||
				input.otherCertifications !== undefined ||
				input.awards !== undefined;

			if (hasCertificationUpdates) {
				const currentCerts = profile.getCertifications();
				profile.updateCertifications({
					hasAbracsMember:
						input.hasAbracsMember ?? currentCerts.hasAbracsMember,
					hasCivilLiabilityInsurance:
						input.hasCivilLiabilityInsurance ??
						currentCerts.hasCivilLiabilityInsurance,
					hasLegalAdvice: input.hasLegalAdvice ?? currentCerts.hasLegalAdvice,
					hasAccountingAdvice:
						input.hasAccountingAdvice ?? currentCerts.hasAccountingAdvice,
					hasWorkSafetyAdvice:
						input.hasWorkSafetyAdvice ?? currentCerts.hasWorkSafetyAdvice,
					hasLgpdCompliance:
						input.hasLgpdCompliance ?? currentCerts.hasLgpdCompliance,
					hasNr1Compliance:
						input.hasNr1Compliance ?? currentCerts.hasNr1Compliance,
					hasComplianceProgram:
						input.hasComplianceProgram ?? currentCerts.hasComplianceProgram,
					hasReclameAquiSeal:
						input.hasReclameAquiSeal ?? currentCerts.hasReclameAquiSeal,
					otherCertifications:
						input.otherCertifications ?? currentCerts.otherCertifications,
					awards: input.awards ?? currentCerts.awards,
				});
			}

			if (input.miniBio !== undefined) {
				profile.updateMiniBio(input.miniBio);
			}

			if (input.educationAndCertifications !== undefined) {
				profile.updateEducationAndCertifications(
					input.educationAndCertifications,
				);
			}

			// 4. Persistir
			await this.deps.syndicProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de síndico atualizado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateDocumentIsUnique(document: string): Promise<void> {
		const existingProfile =
			await this.deps.syndicProfileRepository.findByDocument(document);
		if (existingProfile) {
			throw new BusinessError(
				"Este documento já está cadastrado",
				"DOCUMENT_ALREADY_EXISTS",
			);
		}
	}

	// --- Response ---

	private buildOutput(profile: SyndicProfile): SyndicProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/update-enterprise-profile.use-case.ts =====
import type {
	EnterpriseProfileOutput,
	UpdateEnterpriseProfile,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import type { EnterpriseProfile } from "../../domain/entities/enterprise.entity";
import type { IEnterpriseProfileRepository } from "../../domain/repositories/enterprise.repository";

interface UpdateEnterpriseProfileUseCaseDependencies
	extends UseCaseDependencies {
	enterpriseProfileRepository: IEnterpriseProfileRepository;
}

export class UpdateEnterpriseProfileUseCase extends TransactionalUseCase<
	UpdateEnterpriseProfile,
	EnterpriseProfileOutput
> {
	constructor(
		private readonly deps: UpdateEnterpriseProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: UpdateEnterpriseProfile,
	): Promise<EnterpriseProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando atualização de perfil de empresa",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Buscar perfil existente
			const profile = await this.deps.enterpriseProfileRepository.findByUserId(
				input.userId,
			);
			if (!profile) {
				throw new NotFoundError("Perfil de empresa não encontrado");
			}

			// 2. Validar CNPJ se estiver sendo alterado
			if (input.cnpj && input.cnpj !== profile.getCnpj()) {
				await this.validateCnpjIsUnique(input.cnpj);
			}

			// 3. Atualizar campos
			if (input.logoUrl !== undefined) {
				profile.updateLogo(input.logoUrl);
			}

			if (input.address) {
				profile.updateAddress(input.address);
			}

			if (
				input.financeContactName !== undefined ||
				input.financeContactPhone !== undefined ||
				input.financeContactEmail !== undefined
			) {
				const currentFinance = profile.getFinanceContact();
				profile.updateFinanceContact({
					name: input.financeContactName ?? currentFinance.name,
					phone: input.financeContactPhone ?? currentFinance.phone,
					email: input.financeContactEmail ?? currentFinance.email,
				});
			}

			if (input.operatingCities !== undefined) {
				profile.updateOperatingCities(input.operatingCities);
			}

			// Atualizar certifications
			const hasCertificationUpdates =
				input.hasCivilLiabilityInsurance !== undefined ||
				input.hasLegalAdvice !== undefined ||
				input.hasAccountingAdvice !== undefined ||
				input.hasWorkSafetyAdvice !== undefined ||
				input.hasTechnicalManager !== undefined ||
				input.hasProfessionalCouncilRegularity !== undefined ||
				input.hasLgpdCompliance !== undefined ||
				input.hasNr1Compliance !== undefined ||
				input.hasComplianceProgram !== undefined;

			if (hasCertificationUpdates) {
				const currentCerts = profile.getCertifications();
				profile.updateCertifications({
					hasCivilLiabilityInsurance:
						input.hasCivilLiabilityInsurance ??
						currentCerts.hasCivilLiabilityInsurance,
					hasLegalAdvice: input.hasLegalAdvice ?? currentCerts.hasLegalAdvice,
					hasAccountingAdvice:
						input.hasAccountingAdvice ?? currentCerts.hasAccountingAdvice,
					hasWorkSafetyAdvice:
						input.hasWorkSafetyAdvice ?? currentCerts.hasWorkSafetyAdvice,
					hasTechnicalManager:
						input.hasTechnicalManager ?? currentCerts.hasTechnicalManager,
					hasProfessionalCouncilRegularity:
						input.hasProfessionalCouncilRegularity ??
						currentCerts.hasProfessionalCouncilRegularity,
					hasLgpdCompliance:
						input.hasLgpdCompliance ?? currentCerts.hasLgpdCompliance,
					hasNr1Compliance:
						input.hasNr1Compliance ?? currentCerts.hasNr1Compliance,
					hasComplianceProgram:
						input.hasComplianceProgram ?? currentCerts.hasComplianceProgram,
				});
			}

			// Atualizar ISOs
			const hasIsoUpdates =
				input.hasIso9001 !== undefined ||
				input.hasIso14001 !== undefined ||
				input.hasIso45001 !== undefined ||
				input.hasIso37001 !== undefined ||
				input.hasIso19600_37301 !== undefined;

			if (hasIsoUpdates) {
				const currentIsos = profile.getIsos();
				profile.updateIsos({
					hasIso9001: input.hasIso9001 ?? currentIsos.hasIso9001,
					hasIso14001: input.hasIso14001 ?? currentIsos.hasIso14001,
					hasIso45001: input.hasIso45001 ?? currentIsos.hasIso45001,
					hasIso37001: input.hasIso37001 ?? currentIsos.hasIso37001,
					hasIso19600_37301:
						input.hasIso19600_37301 ?? currentIsos.hasIso19600_37301,
				});
			}

			// Atualizar Seals
			const hasSealUpdates =
				input.hasEsg !== undefined ||
				input.hasGreenSeal !== undefined ||
				input.hasChildFriendlySeal !== undefined ||
				input.hasCarbonFreeSeal !== undefined ||
				input.hasEuRecicloSeal !== undefined ||
				input.hasReclameAquiSeal !== undefined ||
				input.hasCipa !== undefined;

			if (hasSealUpdates) {
				const currentSeals = profile.getSeals();
				profile.updateSeals({
					hasEsg: input.hasEsg ?? currentSeals.hasEsg,
					hasGreenSeal: input.hasGreenSeal ?? currentSeals.hasGreenSeal,
					hasChildFriendlySeal:
						input.hasChildFriendlySeal ?? currentSeals.hasChildFriendlySeal,
					hasCarbonFreeSeal:
						input.hasCarbonFreeSeal ?? currentSeals.hasCarbonFreeSeal,
					hasEuRecicloSeal:
						input.hasEuRecicloSeal ?? currentSeals.hasEuRecicloSeal,
					hasReclameAquiSeal:
						input.hasReclameAquiSeal ?? currentSeals.hasReclameAquiSeal,
					hasCipa: input.hasCipa ?? currentSeals.hasCipa,
				});
			}

			if (input.institutionalDescription !== undefined) {
				profile.updateInstitutionalDescription(input.institutionalDescription);
			}

			// 4. Persistir
			await this.deps.enterpriseProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de empresa atualizado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateCnpjIsUnique(cnpj: string): Promise<void> {
		const existingProfile =
			await this.deps.enterpriseProfileRepository.findByCnpj(cnpj);
		if (existingProfile) {
			throw new BusinessError(
				"Este CNPJ já está cadastrado",
				"CNPJ_ALREADY_EXISTS",
			);
		}
	}

	// --- Response ---

	private buildOutput(profile: EnterpriseProfile): EnterpriseProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/update-resident-profile.use-case.ts =====
import type {
	ResidentProfileOutput,
	UpdateResidentProfile,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import type { ResidentProfile } from "../../domain/entities/resident.entity";
import type { IResidentProfileRepository } from "../../domain/repositories/resident.repository";

interface UpdateResidentProfileUseCaseDependencies extends UseCaseDependencies {
	residentProfileRepository: IResidentProfileRepository;
}

export class UpdateResidentProfileUseCase extends TransactionalUseCase<
	UpdateResidentProfile,
	ResidentProfileOutput
> {
	constructor(private readonly deps: UpdateResidentProfileUseCaseDependencies) {
		super(deps);
	}

	async execute(input: UpdateResidentProfile): Promise<ResidentProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando atualização de perfil de morador",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Buscar perfil existente
			const profile = await this.deps.residentProfileRepository.findByUserId(
				input.userId,
			);
			if (!profile) {
				throw new NotFoundError("Perfil de morador não encontrado");
			}

			// 2. Validar CPF se estiver sendo alterado
			if (input.cpf && input.cpf !== profile.getCpf()) {
				await this.validateCpfIsUnique(input.cpf);
			}

			// 3. Atualizar campos
			if (input.address) {
				profile.updateAddress({
					street: input.address.street,
					number: input.address.number,
					block: input.address.block,
					district: input.address.district,
					city: input.address.city,
					state: input.address.state,
					zipCode: input.address.zipCode,
				});
			}

			if (input.buildingNameOrCode !== undefined) {
				profile.updateBuildingLink(input.buildingNameOrCode);
			}

			// 4. Persistir
			await this.deps.residentProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de morador atualizado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateCpfIsUnique(cpf: string): Promise<void> {
		const existingProfile =
			await this.deps.residentProfileRepository.findByCpf(cpf);
		if (existingProfile) {
			throw new BusinessError(
				"Este CPF já está cadastrado",
				"CPF_ALREADY_EXISTS",
			);
		}
	}

	// --- Response ---

	private buildOutput(profile: ResidentProfile): ResidentProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/get-profile-by-user-id.use-case.ts =====
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { NotFoundError } from "../../../../core/errors";
import type { BaseProfile } from "../../domain/entities/base-profile.entity";
import type { IEnterpriseProfileRepository } from "../../domain/repositories/enterprise.repository";
import type { ILocalCompanyProfileRepository } from "../../domain/repositories/local-company.repository";
import type { IMarketingProfileRepository } from "../../domain/repositories/marketing.repository";
import type { IResidentProfileRepository } from "../../domain/repositories/resident.repository";
import type { ISyndicProfileRepository } from "../../domain/repositories/syndic.repository";

interface GetProfileByUserIdUseCaseDependencies extends UseCaseDependencies {
	residentProfileRepository: IResidentProfileRepository;
	syndicProfileRepository: ISyndicProfileRepository;
	enterpriseProfileRepository: IEnterpriseProfileRepository;
	localCompanyProfileRepository: ILocalCompanyProfileRepository;
	marketingProfileRepository: IMarketingProfileRepository;
}

interface GetProfileByUserIdInput {
	userId: string;
	profileType?:
		| "resident"
		| "syndic"
		| "enterprise"
		| "local-company"
		| "marketing";
}

interface GetProfileByUserIdOutput {
	profile: BaseProfile;
	profileType:
		| "resident"
		| "syndic"
		| "enterprise"
		| "local-company"
		| "marketing";
}

export class GetProfileByUserIdUseCase extends TransactionalUseCase<
	GetProfileByUserIdInput,
	GetProfileByUserIdOutput
> {
	constructor(
		private readonly deps: GetProfileByUserIdUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: GetProfileByUserIdInput,
	): Promise<GetProfileByUserIdOutput> {
		this.logger.info({ userId: input.userId }, "Buscando perfil do usuário");

		// Se profileType foi especificado, buscar apenas naquele repositório
		if (input.profileType) {
			const profile = await this.findProfileByType(
				input.userId,
				input.profileType,
			);
			if (!profile) {
				throw new NotFoundError(
					`Perfil do tipo ${input.profileType} não encontrado para este usuário`,
				);
			}
			return { profile, profileType: input.profileType };
		}

		// Buscar em todos os repositórios (ordem de prioridade)
		const residentProfile =
			await this.deps.residentProfileRepository.findByUserId(input.userId);
		if (residentProfile) {
			return { profile: residentProfile, profileType: "resident" };
		}

		const syndicProfile =
			await this.deps.syndicProfileRepository.findByUserId(input.userId);
		if (syndicProfile) {
			return { profile: syndicProfile, profileType: "syndic" };
		}

		const enterpriseProfile =
			await this.deps.enterpriseProfileRepository.findByUserId(input.userId);
		if (enterpriseProfile) {
			return { profile: enterpriseProfile, profileType: "enterprise" };
		}

		const localCompanyProfile =
			await this.deps.localCompanyProfileRepository.findByUserId(input.userId);
		if (localCompanyProfile) {
			return { profile: localCompanyProfile, profileType: "local-company" };
		}

		const marketingProfile =
			await this.deps.marketingProfileRepository.findByUserId(input.userId);
		if (marketingProfile) {
			return { profile: marketingProfile, profileType: "marketing" };
		}

		// Nenhum perfil encontrado
		throw new NotFoundError("Perfil não encontrado para este usuário");
	}

	private async findProfileByType(
		userId: string,
		profileType: NonNullable<GetProfileByUserIdInput["profileType"]>,
	): Promise<BaseProfile | null> {
		switch (profileType) {
			case "resident":
				return await this.deps.residentProfileRepository.findByUserId(userId);
			case "syndic":
				return await this.deps.syndicProfileRepository.findByUserId(userId);
			case "enterprise":
				return await this.deps.enterpriseProfileRepository.findByUserId(userId);
			case "local-company":
				return await this.deps.localCompanyProfileRepository.findByUserId(
					userId,
				);
			case "marketing":
				return await this.deps.marketingProfileRepository.findByUserId(userId);
		}
	}
}
\n\n===== FILE: ./src/modules/profile/application/use-cases/create-enterprise-profile.use-case.ts =====
import type {
	CreateEnterpriseProfile,
	EnterpriseProfileOutput,
} from "mastersindico-schemas";
import {
	TransactionalUseCase,
	type UseCaseDependencies,
} from "../../../../core/contracts/base-use-case";
import { BusinessError, NotFoundError } from "../../../../core/errors";
import { generateId } from "../../../../shared/utils/id-generator";
import type { IUserRepository } from "../../../user/domain/repositories/user.repository";
import { EnterpriseProfile } from "../../domain/entities/enterprise.entity";
import type { IEnterpriseProfileRepository } from "../../domain/repositories/enterprise.repository";
import { CnpjVO } from "../../domain/value-objects/cnpj.vo";

interface CreateEnterpriseProfileUseCaseDependencies
	extends UseCaseDependencies {
	enterpriseProfileRepository: IEnterpriseProfileRepository;
	userRepository: IUserRepository;
}

export class CreateEnterpriseProfileUseCase extends TransactionalUseCase<
	CreateEnterpriseProfile,
	EnterpriseProfileOutput
> {
	constructor(
		private readonly deps: CreateEnterpriseProfileUseCaseDependencies,
	) {
		super(deps);
	}

	async execute(
		input: CreateEnterpriseProfile,
	): Promise<EnterpriseProfileOutput> {
		this.logger.info(
			{ userId: input.userId },
			"Iniciando criação de perfil de empresa",
		);

		const profile = await this.unitOfWork.run(async () => {
			// 1. Validações de negócio
			await this.validateUserExists(input.userId);
			await this.validateProfileDoesNotExist(input.userId);
			await this.validateCnpjIsUnique(input.cnpj);

			// 2. Criar entidade de perfil
			const profile = this.createProfile(input);

			// 3. Persistir
			await this.deps.enterpriseProfileRepository.save(profile);

			this.logger.info(
				{ profileId: profile.getId(), userId: input.userId },
				"Perfil de empresa criado com sucesso",
			);

			return profile;
		});

		return this.buildOutput(profile);
	}

	// --- Validações ---

	private async validateUserExists(userId: string): Promise<void> {
		const user = await this.deps.userRepository.findById(userId);
		if (!user) {
			throw new NotFoundError("Usuário não encontrado");
		}
	}

	private async validateProfileDoesNotExist(userId: string): Promise<void> {
		const existingProfile =
			await this.deps.enterpriseProfileRepository.findByUserId(userId);
		if (existingProfile) {
			throw new BusinessError(
				"Este usuário já possui um perfil de empresa",
				"PROFILE_ALREADY_EXISTS",
			);
		}
	}

	private async validateCnpjIsUnique(cnpj: string): Promise<void> {
		const existingProfile =
			await this.deps.enterpriseProfileRepository.findByCnpj(cnpj);
		if (existingProfile) {
			throw new BusinessError(
				"Este CNPJ já está cadastrado",
				"CNPJ_ALREADY_EXISTS",
			);
		}
	}

	// --- Criação ---

	private createProfile(input: CreateEnterpriseProfile): EnterpriseProfile {
		// Criar o VO do CNPJ (já validado pelo schema)
		const cnpjVO = CnpjVO.create(input.cnpj);

		return EnterpriseProfile.create({
			id: generateId(),
			userId: input.userId,
			companyName: input.companyName,
			companyTradeName: input.companyTradeName,
			cnpj: cnpjVO,
			foundationDate: input.foundationDate,
			legalRepresentative: input.legalRepresentative,
			commercialContact: input.commercialContact,
			address: input.address,
			financeContact: input.financeContact,
			operatingCities: input.operatingCities.length > 0 ? input.operatingCities : null,
			certifications: {
				hasCivilLiabilityInsurance: input.hasCivilLiabilityInsurance,
				hasLegalAdvice: input.hasLegalAdvice,
				hasAccountingAdvice: input.hasAccountingAdvice,
				hasWorkSafetyAdvice: input.hasWorkSafetyAdvice,
				hasTechnicalManager: input.hasTechnicalManager,
				hasProfessionalCouncilRegularity: input.hasProfessionalCouncilRegularity,
				hasLgpdCompliance: input.hasLgpdCompliance,
				hasNr1Compliance: input.hasNr1Compliance,
				hasComplianceProgram: input.hasComplianceProgram,
			},
			isos: {
				hasIso9001: input.hasIso9001,
				hasIso14001: input.hasIso14001,
				hasIso45001: input.hasIso45001,
				hasIso37001: input.hasIso37001,
				hasIso19600_37301: input.hasIso19600_37301,
			},
			seals: {
				hasEsg: input.hasEsg,
				hasGreenSeal: input.hasGreenSeal,
				hasChildFriendlySeal: input.hasChildFriendlySeal,
				hasCarbonFreeSeal: input.hasCarbonFreeSeal,
				hasEuRecicloSeal: input.hasEuRecicloSeal,
				hasReclameAquiSeal: input.hasReclameAquiSeal,
				hasCipa: input.hasCipa,
			},
			nrs: input.nrs,
			otherCertifications: input.otherCertifications,
			ownCertifications: input.ownCertifications,
			awards: input.awards,
			institutionalDescription: input.institutionalDescription,
			portfolioUrl: input.portfolioUrl,
			status: "incomplete",
			createdAt: new Date(),
			updatedAt: new Date(),
		});
	}

	// --- Response ---

	private buildOutput(profile: EnterpriseProfile): EnterpriseProfileOutput {
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			status: profile.getStatus(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/application/validators/create-local-company-profile.validator.ts =====
import { z } from "zod";
import { cnpjStrictSchema } from "../../../../../../../packages/schemas/src/utils/cnpj.schema";
import { cpfStrictSchema } from "../../../../../../../packages/schemas/src/utils/cpf.schema";

// Schema unificado inteligente (CPF OU CNPJ)
const documentSchema = z
	.string()
	.transform((val) => val.replace(/\D/g, "")) // Limpa
	.superRefine((val, ctx) => {
		if (val.length === 11) {
			const result = cpfStrictSchema.safeParse(val);
			if (!result.success)
				ctx.addIssue({ message: "CPF Inválido", path: ["document"] });
		} else if (val.length === 14) {
			const result = cnpjStrictSchema.safeParse(val);
			if (!result.success)
				ctx.addIssue({ message: "CNPJ Inválido", path: ["document"] });
		} else {
			ctx.addIssue({
				message: "Documento deve ser CPF (11) ou CNPJ (14)",
				path: ["document"],
			});
		}
	});

export const createLocalCompanyProfileSchema = z.object({
	document: documentSchema,
});
\n\n===== FILE: ./src/modules/profile/infrastructure/database/drizzle/repositories/syndic.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { syndicProfile } from "../../../../../../infrastructure/database/drizzle/schema";
import { SyndicProfile } from "../../../../domain/entities/syndic.entity";
import type { ISyndicProfileRepository } from "../../../../domain/repositories/syndic.repository";

export class SyndicProfileRepositoryImpl
	extends BaseRepository
	implements ISyndicProfileRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<SyndicProfile | null> {
		const result = await this.db.query.syndicProfile.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<SyndicProfile | null> {
		const result = await this.db.query.syndicProfile.findFirst({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByDocument(document: string): Promise<SyndicProfile | null> {
		const result = await this.db.query.syndicProfile.findFirst({
			where: {
				document,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(profile: SyndicProfile): Promise<void> {
		const data = this.toPersistence(profile);

		await this.db
			.insert(syndicProfile)
			.values(data)
			.onConflictDoUpdate({
				target: syndicProfile.id,
				set: {
					street: data.street,
					number: data.number,
					block: data.block,
					district: data.district,
					city: data.city,
					state: data.state,
					zipCode: data.zipCode,
					type: data.type,
					experienceYears: data.experienceYears,
					buildingsCount: data.buildingsCount,
					operatingCities: data.operatingCities,
					hasAbracsMember: data.hasAbracsMember,
					hasCivilLiabilityInsurance: data.hasCivilLiabilityInsurance,
					hasLegalAdvice: data.hasLegalAdvice,
					hasAccountingAdvice: data.hasAccountingAdvice,
					hasWorkSafetyAdvice: data.hasWorkSafetyAdvice,
					hasLgpdCompliance: data.hasLgpdCompliance,
					hasNr1Compliance: data.hasNr1Compliance,
					hasComplianceProgram: data.hasComplianceProgram,
					hasReclameAquiSeal: data.hasReclameAquiSeal,
					otherCertifications: data.otherCertifications,
					awards: data.awards,
					miniBio: data.miniBio,
					educationAndCertifications: data.educationAndCertifications,
					linkedAdministrators: data.linkedAdministrators,
					status: data.status,
					updatedAt: new Date(),
					deletedAt: data.deletedAt,
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(syndicProfile)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(syndicProfile.id, id));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(syndicProfile)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(syndicProfile.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(syndicProfile).where(eq(syndicProfile.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof syndicProfile.$inferSelect): SyndicProfile {
		return SyndicProfile.fromPersistence({
			id: raw.id,
			userId: raw.userId,
			birthDate: raw.birthDate,
			document: raw.document,
			street: raw.street,
			number: raw.number,
			block: raw.block,
			district: raw.district,
			city: raw.city,
			state: raw.state,
			zipCode: raw.zipCode,
			type: raw.type,
			experienceYears: raw.experienceYears,
			buildingsCount: raw.buildingsCount,
			operatingCities: raw.operatingCities,
			hasAbracsMember: raw.hasAbracsMember,
			hasCivilLiabilityInsurance: raw.hasCivilLiabilityInsurance,
			hasLegalAdvice: raw.hasLegalAdvice,
			hasAccountingAdvice: raw.hasAccountingAdvice,
			hasWorkSafetyAdvice: raw.hasWorkSafetyAdvice,
			hasLgpdCompliance: raw.hasLgpdCompliance,
			hasNr1Compliance: raw.hasNr1Compliance,
			hasComplianceProgram: raw.hasComplianceProgram,
			hasReclameAquiSeal: raw.hasReclameAquiSeal,
			otherCertifications: raw.otherCertifications,
			awards: raw.awards,
			miniBio: raw.miniBio,
			educationAndCertifications: raw.educationAndCertifications,
			linkedAdministrators: raw.linkedAdministrators,
			status: raw.status,
			createdAt: raw.createdAt,
			updatedAt: raw.updatedAt,
			deletedAt: raw.deletedAt,
		});
	}

	private toPersistence(
		profile: SyndicProfile,
	): typeof syndicProfile.$inferInsert {
		const dto = profile.toDto();
		const certifications = profile.getCertifications();

		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			birthDate: profile.getBirthDate(),
			document: profile.getDocument(),
			street: dto.street,
			number: dto.number,
			block: dto.block,
			district: dto.district,
			city: dto.city,
			state: dto.state,
			zipCode: dto.zipCode,
			type: profile.getType(),
			experienceYears: profile.getExperienceYears(),
			buildingsCount: profile.getBuildingsCount(),
			operatingCities: profile.getOperatingCities(),
			hasAbracsMember: certifications.hasAbracsMember,
			hasCivilLiabilityInsurance: certifications.hasCivilLiabilityInsurance,
			hasLegalAdvice: certifications.hasLegalAdvice,
			hasAccountingAdvice: certifications.hasAccountingAdvice,
			hasWorkSafetyAdvice: certifications.hasWorkSafetyAdvice,
			hasLgpdCompliance: certifications.hasLgpdCompliance,
			hasNr1Compliance: certifications.hasNr1Compliance,
			hasComplianceProgram: certifications.hasComplianceProgram,
			hasReclameAquiSeal: certifications.hasReclameAquiSeal,
			otherCertifications: certifications.otherCertifications,
			awards: certifications.awards,
			miniBio: profile.getMiniBio(),
			educationAndCertifications: profile.getEducationAndCertifications(),
			linkedAdministrators: profile.getLinkedAdministrators(),
			status: profile.getStatus(),
			createdAt: profile.getCreatedAt(),
			updatedAt: profile.getUpdatedAt(),
			deletedAt: profile.getDeletedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/infrastructure/database/drizzle/repositories/resident.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { residentProfile } from "../../../../../../infrastructure/database/drizzle/schema";
import { ResidentProfile } from "../../../../domain/entities/resident.entity";
import type { IResidentProfileRepository } from "../../../../domain/repositories/resident.repository";

export class ResidentProfileRepositoryImpl
	extends BaseRepository
	implements IResidentProfileRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<ResidentProfile | null> {
		const result = await this.db.query.residentProfile.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<ResidentProfile | null> {
		const result = await this.db.query.residentProfile.findFirst({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByCpf(cpf: string): Promise<ResidentProfile | null> {
		const result = await this.db.query.residentProfile.findFirst({
			where: {
				cpf,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(profile: ResidentProfile): Promise<void> {
		const data = this.toPersistence(profile);

		await this.db
			.insert(residentProfile)
			.values(data)
			.onConflictDoUpdate({
				target: residentProfile.id,
				set: {
					street: data.street,
					number: data.number,
					block: data.block,
					district: data.district,
					city: data.city,
					state: data.state,
					zipCode: data.zipCode,
					buildingNameOrCode: data.buildingNameOrCode,
					status: data.status,
					updatedAt: new Date(),
					deletedAt: data.deletedAt,
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(residentProfile)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(residentProfile.id, id));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(residentProfile)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(residentProfile.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(residentProfile).where(eq(residentProfile.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof residentProfile.$inferSelect): ResidentProfile {
		return ResidentProfile.fromPersistence({
			id: raw.id,
			userId: raw.userId,
			birthDate: raw.birthDate,
			cpf: raw.cpf,
			street: raw.street,
			number: raw.number,
			block: raw.block,
			district: raw.district,
			city: raw.city,
			state: raw.state,
			zipCode: raw.zipCode,
			buildingNameOrCode: raw.buildingNameOrCode,
			status: raw.status,
			createdAt: raw.createdAt,
			updatedAt: raw.updatedAt,
			deletedAt: raw.deletedAt,
		});
	}

	private toPersistence(
		profile: ResidentProfile,
	): typeof residentProfile.$inferInsert {
		const dto = profile.toDto();
		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			birthDate: profile.getBirthDate(),
			cpf: profile.getCpf(),
			street: dto.street,
			number: dto.number,
			block: dto.block,
			district: dto.district,
			city: dto.city,
			state: dto.state,
			zipCode: dto.zipCode,
			buildingNameOrCode: profile.getBuildingNameOrCode(),
			status: profile.getStatus(),
			createdAt: profile.getCreatedAt(),
			updatedAt: profile.getUpdatedAt(),
			deletedAt: profile.getDeletedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/infrastructure/database/drizzle/repositories/enterprise.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { enterpriseProfile } from "../../../../../../infrastructure/database/drizzle/schema";
import { EnterpriseProfile } from "../../../../domain/entities/enterprise.entity";
import type { IEnterpriseProfileRepository } from "../../../../domain/repositories/enterprise.repository";

export class EnterpriseProfileRepositoryImpl
	extends BaseRepository
	implements IEnterpriseProfileRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<EnterpriseProfile | null> {
		const result = await this.db.query.enterpriseProfile.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<EnterpriseProfile | null> {
		const result = await this.db.query.enterpriseProfile.findFirst({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByCnpj(cnpj: string): Promise<EnterpriseProfile | null> {
		const result = await this.db.query.enterpriseProfile.findFirst({
			where: {
				cnpj,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(profile: EnterpriseProfile): Promise<void> {
		const data = this.toPersistence(profile);

		await this.db
			.insert(enterpriseProfile)
			.values(data)
			.onConflictDoUpdate({
				target: enterpriseProfile.id,
				set: {
					companyName: data.companyName,
					companyTradeName: data.companyTradeName,
					legalRepresentativeName: data.legalRepresentativeName,
					legalRepresentativeEmail: data.legalRepresentativeEmail,
					legalRepresentativePhone: data.legalRepresentativePhone,
					commercialEmail: data.commercialEmail,
					commercialPhone: data.commercialPhone,
					street: data.street,
					number: data.number,
					block: data.block,
					district: data.district,
					city: data.city,
					state: data.state,
					zipCode: data.zipCode,
					financeContactName: data.financeContactName,
					financeContactPhone: data.financeContactPhone,
					financeContactEmail: data.financeContactEmail,
					operatingCities: data.operatingCities,
					hasCivilLiabilityInsurance: data.hasCivilLiabilityInsurance,
					hasLegalAdvice: data.hasLegalAdvice,
					hasAccountingAdvice: data.hasAccountingAdvice,
					hasWorkSafetyAdvice: data.hasWorkSafetyAdvice,
					hasTechnicalManager: data.hasTechnicalManager,
					hasProfessionalCouncilRegularity: data.hasProfessionalCouncilRegularity,
					hasLgpdCompliance: data.hasLgpdCompliance,
					hasNr1Compliance: data.hasNr1Compliance,
					hasComplianceProgram: data.hasComplianceProgram,
					hasIso9001: data.hasIso9001,
					hasIso14001: data.hasIso14001,
					hasIso45001: data.hasIso45001,
					hasIso37001: data.hasIso37001,
					hasIso19600_37301: data.hasIso19600_37301,
					hasEsg: data.hasEsg,
					hasGreenSeal: data.hasGreenSeal,
					hasChildFriendlySeal: data.hasChildFriendlySeal,
					hasCarbonFreeSeal: data.hasCarbonFreeSeal,
					hasEuRecicloSeal: data.hasEuRecicloSeal,
					hasReclameAquiSeal: data.hasReclameAquiSeal,
					hasCipa: data.hasCipa,
					nrs: data.nrs,
					otherCertifications: data.otherCertifications,
					ownCertifications: data.ownCertifications,
					awards: data.awards,
					institutionalDescription: data.institutionalDescription,
					portfolioUrl: data.portfolioUrl,
					status: data.status,
					updatedAt: new Date(),
					deletedAt: data.deletedAt,
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(enterpriseProfile)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(enterpriseProfile.id, id));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(enterpriseProfile)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(enterpriseProfile.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(enterpriseProfile).where(eq(enterpriseProfile.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof enterpriseProfile.$inferSelect): EnterpriseProfile {
		return EnterpriseProfile.fromPersistence({
			id: raw.id,
			userId: raw.userId,
			companyName: raw.companyName,
			companyTradeName: raw.companyTradeName,
			cnpj: raw.cnpj,
			foundationDate: raw.foundationDate,
			legalRepresentativeName: raw.legalRepresentativeName,
			legalRepresentativeEmail: raw.legalRepresentativeEmail,
			legalRepresentativePhone: raw.legalRepresentativePhone,
			commercialEmail: raw.commercialEmail,
			commercialPhone: raw.commercialPhone,
			street: raw.street,
			number: raw.number,
			block: raw.block,
			district: raw.district,
			city: raw.city,
			state: raw.state,
			zipCode: raw.zipCode,
			financeContactName: raw.financeContactName,
			financeContactPhone: raw.financeContactPhone,
			financeContactEmail: raw.financeContactEmail,
			operatingCities: raw.operatingCities,
			hasCivilLiabilityInsurance: raw.hasCivilLiabilityInsurance,
			hasLegalAdvice: raw.hasLegalAdvice,
			hasAccountingAdvice: raw.hasAccountingAdvice,
			hasWorkSafetyAdvice: raw.hasWorkSafetyAdvice,
			hasTechnicalManager: raw.hasTechnicalManager,
			hasProfessionalCouncilRegularity: raw.hasProfessionalCouncilRegularity,
			hasLgpdCompliance: raw.hasLgpdCompliance,
			hasNr1Compliance: raw.hasNr1Compliance,
			hasComplianceProgram: raw.hasComplianceProgram,
			hasIso9001: raw.hasIso9001,
			hasIso14001: raw.hasIso14001,
			hasIso45001: raw.hasIso45001,
			hasIso37001: raw.hasIso37001,
			hasIso19600_37301: raw.hasIso19600_37301,
			hasEsg: raw.hasEsg,
			hasGreenSeal: raw.hasGreenSeal,
			hasChildFriendlySeal: raw.hasChildFriendlySeal,
			hasCarbonFreeSeal: raw.hasCarbonFreeSeal,
			hasEuRecicloSeal: raw.hasEuRecicloSeal,
			hasReclameAquiSeal: raw.hasReclameAquiSeal,
			hasCipa: raw.hasCipa,
			nrs: raw.nrs,
			otherCertifications: raw.otherCertifications,
			ownCertifications: raw.ownCertifications,
			awards: raw.awards,
			institutionalDescription: raw.institutionalDescription,
			portfolioUrl: raw.portfolioUrl,
			status: raw.status,
			createdAt: raw.createdAt,
			updatedAt: raw.updatedAt,
			deletedAt: raw.deletedAt,
		});
	}

	private toPersistence(
		profile: EnterpriseProfile,
	): typeof enterpriseProfile.$inferInsert {
		const dto = profile.toDto();
		const certifications = profile.getCertifications();
		const isos = profile.getIsos();
		const seals = profile.getSeals();
		const legalRep = profile.getLegalRepresentative();
		const commercialContact = profile.getCommercialContact();

		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			companyName: profile.getCompanyName(),
			companyTradeName: profile.getCompanyTradeName(),
			cnpj: profile.getCnpj(),
			foundationDate: profile.getFoundationDate(),
			legalRepresentativeName: legalRep.name,
			legalRepresentativeEmail: legalRep.email,
			legalRepresentativePhone: legalRep.phone || null,
			commercialEmail: commercialContact.email,
			commercialPhone: commercialContact.phone,
			street: dto.street,
			number: dto.number,
			block: dto.block,
			district: dto.district,
			city: dto.city,
			state: dto.state,
			zipCode: dto.zipCode,
			financeContactName: dto.financeContactName,
			financeContactPhone: dto.financeContactPhone,
			financeContactEmail: dto.financeContactEmail,
			operatingCities: profile.getOperatingCities(),
			hasCivilLiabilityInsurance: certifications.hasCivilLiabilityInsurance,
			hasLegalAdvice: certifications.hasLegalAdvice,
			hasAccountingAdvice: certifications.hasAccountingAdvice,
			hasWorkSafetyAdvice: certifications.hasWorkSafetyAdvice,
			hasTechnicalManager: certifications.hasTechnicalManager,
			hasProfessionalCouncilRegularity: certifications.hasProfessionalCouncilRegularity,
			hasLgpdCompliance: certifications.hasLgpdCompliance,
			hasNr1Compliance: certifications.hasNr1Compliance,
			hasComplianceProgram: certifications.hasComplianceProgram,
			hasIso9001: isos.hasIso9001,
			hasIso14001: isos.hasIso14001,
			hasIso45001: isos.hasIso45001,
			hasIso37001: isos.hasIso37001,
			hasIso19600_37301: isos.hasIso19600_37301,
			hasEsg: seals.hasEsg,
			hasGreenSeal: seals.hasGreenSeal,
			hasChildFriendlySeal: seals.hasChildFriendlySeal,
			hasCarbonFreeSeal: seals.hasCarbonFreeSeal,
			hasEuRecicloSeal: seals.hasEuRecicloSeal,
			hasReclameAquiSeal: seals.hasReclameAquiSeal,
			hasCipa: seals.hasCipa,
			nrs: profile.getNrs(),
			otherCertifications: profile.getOtherCertifications(),
			ownCertifications: profile.getOwnCertifications(),
			awards: profile.getAwards(),
			institutionalDescription: profile.getInstitutionalDescription(),
			portfolioUrl: profile.getPortfolioUrl(),
			status: profile.getStatus(),
			createdAt: profile.getCreatedAt(),
			updatedAt: profile.getUpdatedAt(),
			deletedAt: profile.getDeletedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/infrastructure/database/drizzle/repositories/marketing.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { marketingProfile } from "../../../../../../infrastructure/database/drizzle/schema";
import { MarketingProfile } from "../../../../domain/entities/marketing.entity";
import type { IMarketingProfileRepository } from "../../../../domain/repositories/marketing.repository";

export class MarketingProfileRepositoryImpl
	extends BaseRepository
	implements IMarketingProfileRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<MarketingProfile | null> {
		const result = await this.db.query.marketingProfile.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<MarketingProfile | null> {
		const result = await this.db.query.marketingProfile.findFirst({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByCnpj(cnpj: string): Promise<MarketingProfile | null> {
		const result = await this.db.query.marketingProfile.findFirst({
			where: {
				cnpj,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(profile: MarketingProfile): Promise<void> {
		const data = this.toPersistence(profile);

		await this.db
			.insert(marketingProfile)
			.values(data)
			.onConflictDoUpdate({
				target: marketingProfile.id,
				set: {
					companyName: data.companyName,
					companyTradeName: data.companyTradeName,
					legalRepresentativeName: data.legalRepresentativeName,
					legalRepresentativeEmail: data.legalRepresentativeEmail,
					legalRepresentativePhone: data.legalRepresentativePhone,
					commercialEmail: data.commercialEmail,
					commercialPhone: data.commercialPhone,
					street: data.street,
					number: data.number,
					block: data.block,
					district: data.district,
					city: data.city,
					state: data.state,
					zipCode: data.zipCode,
					financeContactName: data.financeContactName,
					financeContactPhone: data.financeContactPhone,
					financeContactEmail: data.financeContactEmail,
					operatingCities: data.operatingCities,
					status: data.status,
					updatedAt: new Date(),
					deletedAt: data.deletedAt,
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(marketingProfile)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(marketingProfile.id, id));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(marketingProfile)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(marketingProfile.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(marketingProfile).where(eq(marketingProfile.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof marketingProfile.$inferSelect): MarketingProfile {
		return MarketingProfile.fromPersistence({
			id: raw.id,
			userId: raw.userId,
			companyName: raw.companyName,
			companyTradeName: raw.companyTradeName,
			cnpj: raw.cnpj,
			foundationDate: raw.foundationDate,
			legalRepresentativeName: raw.legalRepresentativeName,
			legalRepresentativeEmail: raw.legalRepresentativeEmail,
			legalRepresentativePhone: raw.legalRepresentativePhone,
			commercialEmail: raw.commercialEmail,
			commercialPhone: raw.commercialPhone,
			street: raw.street,
			number: raw.number,
			block: raw.block,
			district: raw.district,
			city: raw.city,
			state: raw.state,
			zipCode: raw.zipCode,
			financeContactName: raw.financeContactName,
			financeContactPhone: raw.financeContactPhone,
			financeContactEmail: raw.financeContactEmail,
			operatingCities: raw.operatingCities,
			status: raw.status,
			createdAt: raw.createdAt,
			updatedAt: raw.updatedAt,
			deletedAt: raw.deletedAt,
		});
	}

	private toPersistence(
		profile: MarketingProfile,
	): typeof marketingProfile.$inferInsert {
		const dto = profile.toDto();
		const legalRep = profile.getLegalRepresentative();
		const commercialContact = profile.getCommercialContact();

		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			companyName: profile.getCompanyName(),
			companyTradeName: profile.getCompanyTradeName(),
			cnpj: profile.getCnpj(),
			foundationDate: profile.getFoundationDate(),
			legalRepresentativeName: legalRep.name,
			legalRepresentativeEmail: legalRep.email,
			legalRepresentativePhone: legalRep.phone || null,
			commercialEmail: commercialContact.email,
			commercialPhone: commercialContact.phone,
			street: dto.street,
			number: dto.number,
			block: dto.block,
			district: dto.district,
			city: dto.city,
			state: dto.state,
			zipCode: dto.zipCode,
			financeContactName: dto.financeContactName,
			financeContactPhone: dto.financeContactPhone,
			financeContactEmail: dto.financeContactEmail,
			operatingCities: profile.getOperatingCities(),
			status: profile.getStatus(),
			createdAt: profile.getCreatedAt(),
			updatedAt: profile.getUpdatedAt(),
			deletedAt: profile.getDeletedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/infrastructure/database/drizzle/repositories/local-company.repository.impl.ts =====
/** biome-ignore-all lint/complexity/noUselessConstructor: <> */
import { eq } from "drizzle-orm";
import {
	BaseRepository,
	type BaseRepositoryDependencies,
} from "../../../../../../core/contracts/base-repository";
import { localCompanyProfile } from "../../../../../../infrastructure/database/drizzle/schema";
import { LocalCompanyProfile } from "../../../../domain/entities/local-company.entity";
import type { ILocalCompanyProfileRepository } from "../../../../domain/repositories/local-company.repository";

export class LocalCompanyProfileRepositoryImpl
	extends BaseRepository
	implements ILocalCompanyProfileRepository
{
	constructor(deps: BaseRepositoryDependencies) {
		super(deps);
	}

	// ==================== QUERIES ====================
	async findById(id: string): Promise<LocalCompanyProfile | null> {
		const result = await this.db.query.localCompanyProfile.findFirst({
			where: {
				id,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByUserId(userId: string): Promise<LocalCompanyProfile | null> {
		const result = await this.db.query.localCompanyProfile.findFirst({
			where: {
				userId,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	async findByDocument(document: string): Promise<LocalCompanyProfile | null> {
		const result = await this.db.query.localCompanyProfile.findFirst({
			where: {
				document,
				deletedAt: { isNull: true },
			},
		});

		return result ? this.toDomain(result) : null;
	}

	// ==================== COMMANDS ====================
	async save(profile: LocalCompanyProfile): Promise<void> {
		const data = this.toPersistence(profile);

		await this.db
			.insert(localCompanyProfile)
			.values(data)
			.onConflictDoUpdate({
				target: localCompanyProfile.id,
				set: {
					companyName: data.companyName,
					companyTradeName: data.companyTradeName,
					legalRepresentativeName: data.legalRepresentativeName,
					legalRepresentativeEmail: data.legalRepresentativeEmail,
					legalRepresentativePhone: data.legalRepresentativePhone,
					commercialEmail: data.commercialEmail,
					commercialPhone: data.commercialPhone,
					street: data.street,
					number: data.number,
					block: data.block,
					district: data.district,
					city: data.city,
					state: data.state,
					zipCode: data.zipCode,
					financeContactName: data.financeContactName,
					financeContactPhone: data.financeContactPhone,
					financeContactEmail: data.financeContactEmail,
					photos: data.photos,
					status: data.status,
					updatedAt: new Date(),
					deletedAt: data.deletedAt,
				},
			});
	}

	async softDelete(id: string): Promise<void> {
		await this.db
			.update(localCompanyProfile)
			.set({
				deletedAt: new Date(),
				updatedAt: new Date(),
			})
			.where(eq(localCompanyProfile.id, id));
	}

	async restore(id: string): Promise<void> {
		await this.db
			.update(localCompanyProfile)
			.set({
				deletedAt: null,
				updatedAt: new Date(),
			})
			.where(eq(localCompanyProfile.id, id));
	}

	async hardDelete(id: string): Promise<void> {
		await this.db.delete(localCompanyProfile).where(eq(localCompanyProfile.id, id));
	}

	// ==================== MAPPERS ====================
	private toDomain(raw: typeof localCompanyProfile.$inferSelect): LocalCompanyProfile {
		return LocalCompanyProfile.fromPersistence({
			id: raw.id,
			userId: raw.userId,
			companyName: raw.companyName,
			companyTradeName: raw.companyTradeName,
			document: raw.document,
			foundationDate: raw.foundationDate,
			legalRepresentativeName: raw.legalRepresentativeName,
			legalRepresentativeEmail: raw.legalRepresentativeEmail,
			legalRepresentativePhone: raw.legalRepresentativePhone,
			commercialEmail: raw.commercialEmail,
			commercialPhone: raw.commercialPhone,
			street: raw.street,
			number: raw.number,
			block: raw.block,
			district: raw.district,
			city: raw.city,
			state: raw.state,
			zipCode: raw.zipCode,
			financeContactName: raw.financeContactName,
			financeContactPhone: raw.financeContactPhone,
			financeContactEmail: raw.financeContactEmail,
			photos: raw.photos,
			status: raw.status,
			createdAt: raw.createdAt,
			updatedAt: raw.updatedAt,
			deletedAt: raw.deletedAt,
		});
	}

	private toPersistence(
		profile: LocalCompanyProfile,
	): typeof localCompanyProfile.$inferInsert {
		const dto = profile.toDto();
		const legalRep = profile.getLegalRepresentative();
		const commercialContact = profile.getCommercialContact();

		return {
			id: profile.getId(),
			userId: profile.getUserId(),
			companyName: profile.getCompanyName(),
			companyTradeName: profile.getCompanyTradeName(),
			document: profile.getDocument(),
			foundationDate: profile.getFoundationDate(),
			legalRepresentativeName: legalRep.name,
			legalRepresentativeEmail: legalRep.email,
			legalRepresentativePhone: legalRep.phone || null,
			commercialEmail: commercialContact.email,
			commercialPhone: commercialContact.phone,
			street: dto.street,
			number: dto.number,
			block: dto.block,
			district: dto.district,
			city: dto.city,
			state: dto.state,
			zipCode: dto.zipCode,
			financeContactName: dto.financeContactName,
			financeContactPhone: dto.financeContactPhone,
			financeContactEmail: dto.financeContactEmail,
			photos: profile.getPhotos(),
			status: profile.getStatus(),
			createdAt: profile.getCreatedAt(),
			updatedAt: profile.getUpdatedAt(),
			deletedAt: profile.getDeletedAt(),
		};
	}
}
\n\n===== FILE: ./src/modules/profile/infrastructure/http/routes/profile.routes.ts =====
import type { FastifyInstance } from "fastify";
import type { ZodTypeProvider } from "fastify-type-provider-zod";
import {
	createEnterpriseProfileSchema,
	createLocalCompanyProfileSchema,
	createMarketingProfileSchema,
	createResidentProfileSchema,
	createSyndicProfileSchema,
	enterpriseProfileOutputSchema,
	localCompanyProfileOutputSchema,
	marketingProfileOutputSchema,
	residentProfileOutputSchema,
	syndicProfileOutputSchema,
	updateEnterpriseProfileSchema,
	updateLocalCompanyProfileSchema,
	updateMarketingProfileSchema,
	updateResidentProfileSchema,
	updateSyndicProfileSchema,
} from "mastersindico-schemas";
import { success } from "../../../../../shared/helpers/response.helper";
import { authenticate } from "../../../../../shared/hooks/authenticate.hook";
import {
	apiErrorSchema,
	apiSuccessSchema,
} from "../../../../../shared/types/api-response";

const PROFILE_RATE_LIMITS = {
	create: { max: 10, timeWindow: "1 minute" },
	update: { max: 20, timeWindow: "1 minute" },
	get: { max: 30, timeWindow: "1 minute" },
} as const;

export async function profileRoutes(app: FastifyInstance) {
	const typedApp = app.withTypeProvider<ZodTypeProvider>();

	// ========================================
	// RESIDENT PROFILE ROUTES
	// ========================================

	typedApp.post(
		"/resident",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.create,
			},
			schema: {
				tags: ["Profiles - Resident"],
				body: createResidentProfileSchema,
				summary: "Criar perfil de morador",
				description:
					"Cria um novo perfil de morador vinculado ao usuário autenticado. Requer informações básicas como CPF, data de nascimento e endereço.",
				response: {
					201: apiSuccessSchema(residentProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					409: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { createResidentProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await createResidentProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.status(201).send(success(output));
		},
	);

	typedApp.get(
		"/resident",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.get,
			},
			schema: {
				tags: ["Profiles - Resident"],
				summary: "Obter perfil de morador",
				description:
					"Retorna o perfil de morador do usuário autenticado, se existir.",
				response: {
					200: apiSuccessSchema(residentProfileOutputSchema),
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { getProfileByUserIdUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await getProfileByUserIdUseCase.execute({
				userId,
				profileType: "resident",
			});

			return reply.send(success(output));
		},
	);

	typedApp.put(
		"/resident",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.update,
			},
			schema: {
				tags: ["Profiles - Resident"],
				body: updateResidentProfileSchema,
				summary: "Atualizar perfil de morador",
				description:
					"Atualiza as informações do perfil de morador do usuário autenticado.",
				response: {
					200: apiSuccessSchema(residentProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { updateResidentProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await updateResidentProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.send(success(output));
		},
	);

	// ========================================
	// SYNDIC PROFILE ROUTES
	// ========================================

	typedApp.post(
		"/syndic",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.create,
			},
			schema: {
				tags: ["Profiles - Syndic"],
				body: createSyndicProfileSchema,
				summary: "Criar perfil de síndico",
				description:
					"Cria um novo perfil de síndico vinculado ao usuário autenticado. Requer informações como documento (CPF ou CNPJ), tipo (profissional/morador), experiência e certificações.",
				response: {
					201: apiSuccessSchema(syndicProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					409: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { createSyndicProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await createSyndicProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.status(201).send(success(output));
		},
	);

	typedApp.get(
		"/syndic",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.get,
			},
			schema: {
				tags: ["Profiles - Syndic"],
				summary: "Obter perfil de síndico",
				description:
					"Retorna o perfil de síndico do usuário autenticado, se existir.",
				response: {
					200: apiSuccessSchema(syndicProfileOutputSchema),
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { getProfileByUserIdUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await getProfileByUserIdUseCase.execute({
				userId,
				profileType: "syndic",
			});

			return reply.send(success(output));
		},
	);

	typedApp.put(
		"/syndic",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.update,
			},
			schema: {
				tags: ["Profiles - Syndic"],
				body: updateSyndicProfileSchema,
				summary: "Atualizar perfil de síndico",
				description:
					"Atualiza as informações do perfil de síndico do usuário autenticado.",
				response: {
					200: apiSuccessSchema(syndicProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { updateSyndicProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await updateSyndicProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.send(success(output));
		},
	);

	// ========================================
	// ENTERPRISE PROFILE ROUTES
	// ========================================

	typedApp.post(
		"/enterprise",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.create,
			},
			schema: {
				tags: ["Profiles - Enterprise"],
				body: createEnterpriseProfileSchema,
				summary: "Criar perfil de empresa/condomínio",
				description:
					"Cria um novo perfil de empresa ou condomínio vinculado ao usuário autenticado. Requer informações como CNPJ, dados corporativos, certificações e selos.",
				response: {
					201: apiSuccessSchema(enterpriseProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					409: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { createEnterpriseProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await createEnterpriseProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.status(201).send(success(output));
		},
	);

	typedApp.get(
		"/enterprise",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.get,
			},
			schema: {
				tags: ["Profiles - Enterprise"],
				summary: "Obter perfil de empresa/condomínio",
				description:
					"Retorna o perfil de empresa ou condomínio do usuário autenticado, se existir.",
				response: {
					200: apiSuccessSchema(enterpriseProfileOutputSchema),
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { getProfileByUserIdUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await getProfileByUserIdUseCase.execute({
				userId,
				profileType: "enterprise",
			});

			return reply.send(success(output));
		},
	);

	typedApp.put(
		"/enterprise",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.update,
			},
			schema: {
				tags: ["Profiles - Enterprise"],
				body: updateEnterpriseProfileSchema,
				summary: "Atualizar perfil de empresa/condomínio",
				description:
					"Atualiza as informações do perfil de empresa ou condomínio do usuário autenticado.",
				response: {
					200: apiSuccessSchema(enterpriseProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { updateEnterpriseProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await updateEnterpriseProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.send(success(output));
		},
	);

	// ========================================
	// LOCAL COMPANY PROFILE ROUTES
	// ========================================

	typedApp.post(
		"/local-company",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.create,
			},
			schema: {
				tags: ["Profiles - Local Company"],
				body: createLocalCompanyProfileSchema,
				summary: "Criar perfil de empresa local/prestadora",
				description:
					"Cria um novo perfil de empresa local ou prestadora de serviços vinculado ao usuário autenticado. Aceita CPF ou CNPJ.",
				response: {
					201: apiSuccessSchema(localCompanyProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					409: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { createLocalCompanyProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await createLocalCompanyProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.status(201).send(success(output));
		},
	);

	typedApp.get(
		"/local-company",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.get,
			},
			schema: {
				tags: ["Profiles - Local Company"],
				summary: "Obter perfil de empresa local/prestadora",
				description:
					"Retorna o perfil de empresa local ou prestadora do usuário autenticado, se existir.",
				response: {
					200: apiSuccessSchema(localCompanyProfileOutputSchema),
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { getProfileByUserIdUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await getProfileByUserIdUseCase.execute({
				userId,
				profileType: "local-company",
			});

			return reply.send(success(output));
		},
	);

	typedApp.put(
		"/local-company",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.update,
			},
			schema: {
				tags: ["Profiles - Local Company"],
				body: updateLocalCompanyProfileSchema,
				summary: "Atualizar perfil de empresa local/prestadora",
				description:
					"Atualiza as informações do perfil de empresa local ou prestadora do usuário autenticado.",
				response: {
					200: apiSuccessSchema(localCompanyProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { updateLocalCompanyProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await updateLocalCompanyProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.send(success(output));
		},
	);

	// ========================================
	// MARKETING PROFILE ROUTES
	// ========================================

	typedApp.post(
		"/marketing",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.create,
			},
			schema: {
				tags: ["Profiles - Marketing"],
				body: createMarketingProfileSchema,
				summary: "Criar perfil de agência de marketing",
				description:
					"Cria um novo perfil de agência de marketing vinculado ao usuário autenticado. Requer CNPJ e informações corporativas.",
				response: {
					201: apiSuccessSchema(marketingProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					409: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { createMarketingProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await createMarketingProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.status(201).send(success(output));
		},
	);

	typedApp.get(
		"/marketing",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.get,
			},
			schema: {
				tags: ["Profiles - Marketing"],
				summary: "Obter perfil de agência de marketing",
				description:
					"Retorna o perfil de agência de marketing do usuário autenticado, se existir.",
				response: {
					200: apiSuccessSchema(marketingProfileOutputSchema),
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { getProfileByUserIdUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await getProfileByUserIdUseCase.execute({
				userId,
				profileType: "marketing",
			});

			return reply.send(success(output));
		},
	);

	typedApp.put(
		"/marketing",
		{
			config: {
				rateLimit: PROFILE_RATE_LIMITS.update,
			},
			schema: {
				tags: ["Profiles - Marketing"],
				body: updateMarketingProfileSchema,
				summary: "Atualizar perfil de agência de marketing",
				description:
					"Atualiza as informações do perfil de agência de marketing do usuário autenticado.",
				response: {
					200: apiSuccessSchema(marketingProfileOutputSchema),
					400: apiErrorSchema,
					401: apiErrorSchema,
					404: apiErrorSchema,
				},
			},
			preHandler: [authenticate],
		},
		async (request, reply) => {
			const { updateMarketingProfileUseCase } = request.diScope.cradle;
			const userId = request.user.getId();

			const output = await updateMarketingProfileUseCase.execute({
				...request.body,
				userId,
			});

			return reply.send(success(output));
		},
	);
}
\n\n===== FILE: ./src/modules/profile/profile.module.ts =====
import { diContainer } from "@fastify/awilix";
import { asClass, Lifetime } from "awilix";
import type { FastifyInstance } from "fastify";
import type { Module } from "../../shared/module/module.interface";
import { CreateEnterpriseProfileUseCase } from "./application/use-cases/create-enterprise-profile.use-case";
import { CreateLocalCompanyProfileUseCase } from "./application/use-cases/create-local-company-profile.use-case";
import { CreateMarketingProfileUseCase } from "./application/use-cases/create-marketing-profile.use-case";
import { CreateResidentProfileUseCase } from "./application/use-cases/create-resident-profile.use-case";
import { CreateSyndicProfileUseCase } from "./application/use-cases/create-syndic-profile.use-case";
import { GetProfileByUserIdUseCase } from "./application/use-cases/get-profile-by-user-id.use-case";
import { UpdateEnterpriseProfileUseCase } from "./application/use-cases/update-enterprise-profile.use-case";
import { UpdateLocalCompanyProfileUseCase } from "./application/use-cases/update-local-company-profile.use-case";
import { UpdateMarketingProfileUseCase } from "./application/use-cases/update-marketing-profile.use-case";
import { UpdateResidentProfileUseCase } from "./application/use-cases/update-resident-profile.use-case";
import { UpdateSyndicProfileUseCase } from "./application/use-cases/update-syndic-profile.use-case";
import { EnterpriseProfileRepositoryImpl } from "./infrastructure/database/drizzle/repositories/enterprise.repository.impl";
import { LocalCompanyProfileRepositoryImpl } from "./infrastructure/database/drizzle/repositories/local-company.repository.impl";
import { MarketingProfileRepositoryImpl } from "./infrastructure/database/drizzle/repositories/marketing.repository.impl";
import { ResidentProfileRepositoryImpl } from "./infrastructure/database/drizzle/repositories/resident.repository.impl";
import { SyndicProfileRepositoryImpl } from "./infrastructure/database/drizzle/repositories/syndic.repository.impl";
import { profileRoutes } from "./infrastructure/http/routes/profile.routes";

export class ProfileModule implements Module {
	async register(app: FastifyInstance): Promise<void> {
		diContainer.register({
			// --- REPOSITORIES (SCOPED) ---
			residentProfileRepository: asClass(ResidentProfileRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			syndicProfileRepository: asClass(SyndicProfileRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			enterpriseProfileRepository: asClass(EnterpriseProfileRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			localCompanyProfileRepository: asClass(LocalCompanyProfileRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),
			marketingProfileRepository: asClass(MarketingProfileRepositoryImpl, {
				lifetime: Lifetime.SCOPED,
			}),

			// --- USE CASES (SCOPED) ---
			// Create Use Cases
			createResidentProfileUseCase: asClass(CreateResidentProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			createSyndicProfileUseCase: asClass(CreateSyndicProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			createEnterpriseProfileUseCase: asClass(CreateEnterpriseProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			createLocalCompanyProfileUseCase: asClass(
				CreateLocalCompanyProfileUseCase,
				{
					lifetime: Lifetime.SCOPED,
				},
			),
			createMarketingProfileUseCase: asClass(CreateMarketingProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),

			// Update Use Cases
			updateResidentProfileUseCase: asClass(UpdateResidentProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			updateSyndicProfileUseCase: asClass(UpdateSyndicProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			updateEnterpriseProfileUseCase: asClass(UpdateEnterpriseProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
			updateLocalCompanyProfileUseCase: asClass(
				UpdateLocalCompanyProfileUseCase,
				{
					lifetime: Lifetime.SCOPED,
				},
			),
			updateMarketingProfileUseCase: asClass(UpdateMarketingProfileUseCase, {
				lifetime: Lifetime.SCOPED,
			}),

			// Get Use Case
			getProfileByUserIdUseCase: asClass(GetProfileByUserIdUseCase, {
				lifetime: Lifetime.SCOPED,
			}),
		});

		// Rotas
		await app.register(profileRoutes, { prefix: "/v1/profiles" });

		app.log.info("✅ ProfileModule registered");
	}

	async bootstrap(app: FastifyInstance): Promise<void> {
		app.log.info("✅ ProfileModule bootstrapped");
	}
}
\n\n===== FILE: ./src/modules/billing/domain/entities/plan.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { UserRole } from "../../../user/domain/enums/user-role";
import type { SubscriptionPlan } from "../enums";
import { Money } from "../value-objects/money.value-object";

interface PlanProps {
	id: string;
	type: SubscriptionPlan;
	name: string;
	description: string | null;
	allowedRole: UserRole;
	priceMonthly: Money | null;
	priceYearly: Money | null;
	currency: string;
	trialDays: number | null;
	hasFreeTrial: boolean;
	provider: string;
	providerProductId: string | null;
	providerPriceMonthlyId: string | null;
	providerPriceYearlyId: string | null;
	version: number;
	isActive: boolean;
	isPublic: boolean;
	isDefault: boolean;
	sortOrder: number;
	maxUsers: number | null;
	metadata: Record<string, unknown> | null;
	highlightFeatures: unknown[] | null;
	createdAt: Date;
	updatedAt: Date;
}

export class Plan {
	private readonly id: string;
	private type: SubscriptionPlan;
	private name: string;
	private description: string | null;
	private allowedRole: UserRole;
	private priceMonthly: Money | null;
	private priceYearly: Money | null;
	private currency: string;
	private trialDays: number | null;
	private hasFreeTrial: boolean;
	private provider: string;
	private providerProductId: string | null;
	private providerPriceMonthlyId: string | null;
	private providerPriceYearlyId: string | null;
	private version: number;
	private isActive: boolean;
	private isPublic: boolean;
	private isDefault: boolean;
	private sortOrder: number;
	private maxUsers: number | null;
	private metadata: Record<string, unknown> | null;
	private highlightFeatures: unknown[] | null;
	private readonly createdAt: Date;
	private updatedAt: Date;

	constructor(props: PlanProps) {
		this.id = props.id;
		this.type = props.type;
		this.name = props.name;
		this.description = props.description;
		this.allowedRole = props.allowedRole;
		this.priceMonthly = props.priceMonthly;
		this.priceYearly = props.priceYearly;
		this.currency = props.currency;
		this.trialDays = props.trialDays;
		this.hasFreeTrial = props.hasFreeTrial;
		this.provider = props.provider;
		this.providerProductId = props.providerProductId;
		this.providerPriceMonthlyId = props.providerPriceMonthlyId;
		this.providerPriceYearlyId = props.providerPriceYearlyId;
		this.version = props.version;
		this.isActive = props.isActive;
		this.isPublic = props.isPublic;
		this.isDefault = props.isDefault;
		this.sortOrder = props.sortOrder;
		this.maxUsers = props.maxUsers;
		this.metadata = props.metadata;
		this.highlightFeatures = props.highlightFeatures;
		this.createdAt = props.createdAt;
		this.updatedAt = props.updatedAt;
	}

	getId(): string {
		return this.id;
	}

	getType(): SubscriptionPlan {
		return this.type;
	}

	getName(): string {
		return this.name;
	}

	getDescription(): string | null {
		return this.description;
	}

	getAllowedRole(): UserRole {
		return this.allowedRole;
	}

	getPriceMonthly(): Money | null {
		return this.priceMonthly;
	}

	getPriceYearly(): Money | null {
		return this.priceYearly;
	}

	getCurrency(): string {
		return this.currency;
	}

	getTrialDays(): number | null {
		return this.trialDays;
	}

	getHasFreeTrial(): boolean {
		return this.hasFreeTrial;
	}

	getProvider(): string {
		return this.provider;
	}

	getProviderProductId(): string | null {
		return this.providerProductId;
	}

	getProviderPriceMonthlyId(): string | null {
		return this.providerPriceMonthlyId;
	}

	getProviderPriceYearlyId(): string | null {
		return this.providerPriceYearlyId;
	}

	getVersion(): number {
		return this.version;
	}

	getIsActive(): boolean {
		return this.isActive;
	}

	getIsPublic(): boolean {
		return this.isPublic;
	}

	getIsDefault(): boolean {
		return this.isDefault;
	}

	getSortOrder(): number {
		return this.sortOrder;
	}

	getMaxUsers(): number | null {
		return this.maxUsers;
	}

	getMetadata(): Record<string, unknown> | null {
		return this.metadata ? { ...this.metadata } : null;
	}

	getHighlightFeatures(): unknown[] | null {
		return this.highlightFeatures ? [...this.highlightFeatures] : null;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	isFree(): boolean {
		const monthlyZeroOrNull =
			this.priceMonthly === null || this.priceMonthly.isZero();
		const yearlyZeroOrNull =
			this.priceYearly === null || this.priceYearly.isZero();
		return monthlyZeroOrNull && yearlyZeroOrNull;
	}

	hasTrialPeriod(): boolean {
		return this.hasFreeTrial && this.trialDays !== null && this.trialDays > 0;
	}

	activate(): void {
		if (this.isActive) {
			throw new BusinessError(
				"O plano já está ativo.",
				"PLAN_ALREADY_ACTIVE",
			);
		}
		this.isActive = true;
		this.updatedAt = new Date();
	}

	deactivate(): void {
		if (!this.isActive) {
			throw new BusinessError(
				"O plano já está inativo.",
				"PLAN_ALREADY_INACTIVE",
			);
		}
		this.isActive = false;
		this.updatedAt = new Date();
	}

	updatePrices(
		priceMonthly: Money | null,
		priceYearly: Money | null,
	): void {
		this.priceMonthly = priceMonthly;
		this.priceYearly = priceYearly;
		this.updatedAt = new Date();
	}

	updateTrialDays(days: number | null): void {
		if (days !== null && days < 0) {
			throw new BusinessError(
				"Os dias de trial não podem ser negativos.",
				"INVALID_TRIAL_DAYS",
			);
		}
		this.trialDays = days;
		this.hasFreeTrial = days !== null && days > 0;
		this.updatedAt = new Date();
	}

	updateProviderIds(
		productId: string | null,
		priceMonthlyId: string | null,
		priceYearlyId: string | null,
	): void {
		this.providerProductId = productId;
		this.providerPriceMonthlyId = priceMonthlyId;
		this.providerPriceYearlyId = priceYearlyId;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			id: this.id,
			type: this.type,
			name: this.name,
			description: this.description,
			allowedRole: this.allowedRole,
			priceMonthly: this.priceMonthly?.toCents() ?? null,
			priceYearly: this.priceYearly?.toCents() ?? null,
			currency: this.currency,
			trialDays: this.trialDays,
			hasFreeTrial: this.hasFreeTrial,
			provider: this.provider,
			providerProductId: this.providerProductId,
			providerPriceMonthlyId: this.providerPriceMonthlyId,
			providerPriceYearlyId: this.providerPriceYearlyId,
			version: this.version,
			isActive: this.isActive,
			isPublic: this.isPublic,
			isDefault: this.isDefault,
			sortOrder: this.sortOrder,
			maxUsers: this.maxUsers,
			metadata: this.metadata,
			highlightFeatures: this.highlightFeatures,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}

	static create(props: PlanProps): Plan {
		return new Plan(props);
	}

	static fromPersistence(data: {
		id: string;
		type: SubscriptionPlan;
		name: string;
		description: string | null;
		allowedRole: UserRole;
		priceMonthly: number | null;
		priceYearly: number | null;
		currency: string;
		trialDays: number | null;
		hasFreeTrial: boolean;
		provider: string;
		providerProductId: string | null;
		providerPriceMonthlyId: string | null;
		providerPriceYearlyId: string | null;
		version: number;
		isActive: boolean;
		isPublic: boolean;
		isDefault: boolean;
		sortOrder: number;
		maxUsers: number | null;
		metadata: Record<string, unknown> | null;
		highlightFeatures: unknown[] | null;
		createdAt: Date;
		updatedAt: Date;
	}): Plan {
		return new Plan({
			id: data.id,
			type: data.type,
			name: data.name,
			description: data.description,
			allowedRole: data.allowedRole,
			priceMonthly:
				data.priceMonthly !== null
					? Money.fromCents(data.priceMonthly, data.currency)
					: null,
			priceYearly:
				data.priceYearly !== null
					? Money.fromCents(data.priceYearly, data.currency)
					: null,
			currency: data.currency,
			trialDays: data.trialDays,
			hasFreeTrial: data.hasFreeTrial,
			provider: data.provider,
			providerProductId: data.providerProductId,
			providerPriceMonthlyId: data.providerPriceMonthlyId,
			providerPriceYearlyId: data.providerPriceYearlyId,
			version: data.version,
			isActive: data.isActive,
			isPublic: data.isPublic,
			isDefault: data.isDefault,
			sortOrder: data.sortOrder,
			maxUsers: data.maxUsers,
			metadata: data.metadata,
			highlightFeatures: data.highlightFeatures,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/billing/domain/entities/subscription.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { BillingCycle, CollectionMethod, SubscriptionStatus } from "../enums";
import type { Money } from "../value-objects/money.value-object";

const VALID_TRANSITIONS: Record<SubscriptionStatus, SubscriptionStatus[]> = {
	incomplete: ["active", "incomplete_expired", "canceled"],
	incomplete_expired: [],
	trialing: ["active", "past_due", "canceled"],
	active: ["past_due", "canceled", "paused"],
	past_due: ["active", "canceled", "unpaid"],
	canceled: [],
	unpaid: ["active", "canceled"],
	paused: ["active", "canceled"],
};

interface SubscriptionProps {
	id: string;
	userId: string;
	planId: string;
	status: SubscriptionStatus;
	billingCycle: BillingCycle | null;
	amount: Money | null;
	currency: string;
	currentPeriodStart: Date;
	currentPeriodEnd: Date;
	trialStart: Date | null;
	trialEnd: Date | null;
	startsAt: Date;
	endsAt: Date | null;
	nextBillingDate: Date | null;
	cancelAtPeriodEnd: boolean;
	canceledAt: Date | null;
	cancelReason: string | null;
	cancelFeedback: string | null;
	pausedAt: Date | null;
	resumesAt: Date | null;
	pauseReason: string | null;
	defaultPaymentMethodId: string | null;
	collectionMethod: CollectionMethod | null;
	daysUntilDue: number | null;
	latestInvoiceId: string | null;
	pendingUpdateId: string | null;
	provider: string;
	providerSubscriptionId: string | null;
	providerCustomerId: string | null;
	providerPriceId: string | null;
	metadata: Record<string, unknown> | null;
	createdAt: Date;
	updatedAt: Date;
}

export class Subscription {
	private readonly id: string;
	private readonly userId: string;
	private planId: string;
	private status: SubscriptionStatus;
	private billingCycle: BillingCycle | null;
	private amount: Money | null;
	private currency: string;
	private currentPeriodStart: Date;
	private currentPeriodEnd: Date;
	private trialStart: Date | null;
	private trialEnd: Date | null;
	private startsAt: Date;
	private endsAt: Date | null;
	private nextBillingDate: Date | null;
	private cancelAtPeriodEnd: boolean;
	private canceledAt: Date | null;
	private cancelReason: string | null;
	private cancelFeedback: string | null;
	private pausedAt: Date | null;
	private resumesAt: Date | null;
	private pauseReason: string | null;
	private defaultPaymentMethodId: string | null;
	private collectionMethod: CollectionMethod | null;
	private daysUntilDue: number | null;
	private latestInvoiceId: string | null;
	private pendingUpdateId: string | null;
	private provider: string;
	private providerSubscriptionId: string | null;
	private providerCustomerId: string | null;
	private providerPriceId: string | null;
	private metadata: Record<string, unknown> | null;
	private readonly createdAt: Date;
	private updatedAt: Date;

	constructor(props: SubscriptionProps) {
		this.id = props.id;
		this.userId = props.userId;
		this.planId = props.planId;
		this.status = props.status;
		this.billingCycle = props.billingCycle;
		this.amount = props.amount;
		this.currency = props.currency;
		this.currentPeriodStart = props.currentPeriodStart;
		this.currentPeriodEnd = props.currentPeriodEnd;
		this.trialStart = props.trialStart;
		this.trialEnd = props.trialEnd;
		this.startsAt = props.startsAt;
		this.endsAt = props.endsAt;
		this.nextBillingDate = props.nextBillingDate;
		this.cancelAtPeriodEnd = props.cancelAtPeriodEnd;
		this.canceledAt = props.canceledAt;
		this.cancelReason = props.cancelReason;
		this.cancelFeedback = props.cancelFeedback;
		this.pausedAt = props.pausedAt;
		this.resumesAt = props.resumesAt;
		this.pauseReason = props.pauseReason;
		this.defaultPaymentMethodId = props.defaultPaymentMethodId;
		this.collectionMethod = props.collectionMethod;
		this.daysUntilDue = props.daysUntilDue;
		this.latestInvoiceId = props.latestInvoiceId;
		this.pendingUpdateId = props.pendingUpdateId;
		this.provider = props.provider;
		this.providerSubscriptionId = props.providerSubscriptionId;
		this.providerCustomerId = props.providerCustomerId;
		this.providerPriceId = props.providerPriceId;
		this.metadata = props.metadata;
		this.createdAt = props.createdAt;
		this.updatedAt = props.updatedAt;
	}

	// Getters

	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getPlanId(): string {
		return this.planId;
	}

	getStatus(): SubscriptionStatus {
		return this.status;
	}

	getBillingCycle(): BillingCycle | null {
		return this.billingCycle;
	}

	getAmount(): Money | null {
		return this.amount;
	}

	getCurrency(): string {
		return this.currency;
	}

	getCurrentPeriodStart(): Date {
		return this.currentPeriodStart;
	}

	getCurrentPeriodEnd(): Date {
		return this.currentPeriodEnd;
	}

	getTrialStart(): Date | null {
		return this.trialStart;
	}

	getTrialEnd(): Date | null {
		return this.trialEnd;
	}

	getStartsAt(): Date {
		return this.startsAt;
	}

	getEndsAt(): Date | null {
		return this.endsAt;
	}

	getNextBillingDate(): Date | null {
		return this.nextBillingDate;
	}

	getCancelAtPeriodEnd(): boolean {
		return this.cancelAtPeriodEnd;
	}

	getCanceledAt(): Date | null {
		return this.canceledAt;
	}

	getCancelReason(): string | null {
		return this.cancelReason;
	}

	getCancelFeedback(): string | null {
		return this.cancelFeedback;
	}

	getPausedAt(): Date | null {
		return this.pausedAt;
	}

	getResumesAt(): Date | null {
		return this.resumesAt;
	}

	getPauseReason(): string | null {
		return this.pauseReason;
	}

	getDefaultPaymentMethodId(): string | null {
		return this.defaultPaymentMethodId;
	}

	getCollectionMethod(): CollectionMethod | null {
		return this.collectionMethod;
	}

	getDaysUntilDue(): number | null {
		return this.daysUntilDue;
	}

	getLatestInvoiceId(): string | null {
		return this.latestInvoiceId;
	}

	getPendingUpdateId(): string | null {
		return this.pendingUpdateId;
	}

	getProvider(): string {
		return this.provider;
	}

	getProviderSubscriptionId(): string | null {
		return this.providerSubscriptionId;
	}

	getProviderCustomerId(): string | null {
		return this.providerCustomerId;
	}

	getProviderPriceId(): string | null {
		return this.providerPriceId;
	}

	getMetadata(): Record<string, unknown> | null {
		return this.metadata ? { ...this.metadata } : null;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	// Query methods

	isActive(): boolean {
		return this.status === "active";
	}

	isTrialing(): boolean {
		return this.status === "trialing";
	}

	isPastDue(): boolean {
		return this.status === "past_due";
	}

	isCanceled(): boolean {
		return this.status === "canceled";
	}

	isPaused(): boolean {
		return this.status === "paused";
	}

	isInTrial(): boolean {
		return this.status === "trialing" && this.trialEnd !== null && this.trialEnd > new Date();
	}

	canBeCanceled(): boolean {
		return this.status !== "canceled" && this.status !== "incomplete_expired";
	}

	// State machine

	private transitionTo(newStatus: SubscriptionStatus): void {
		const allowed = VALID_TRANSITIONS[this.status];
		if (!allowed.includes(newStatus)) {
			throw new BusinessError(
				`Transição de status inválida: ${this.status} -> ${newStatus}`,
				"INVALID_SUBSCRIPTION_STATUS_TRANSITION",
			);
		}
		this.status = newStatus;
		this.updatedAt = new Date();
	}

	// Commands

	activate(): void {
		this.transitionTo("active");
	}

	markPastDue(): void {
		this.transitionTo("past_due");
	}

	markUnpaid(): void {
		this.transitionTo("unpaid");
	}

	expire(): void {
		if (this.status !== "incomplete") {
			throw new BusinessError(
				"Somente assinaturas incompletas podem expirar.",
				"SUBSCRIPTION_NOT_INCOMPLETE",
			);
		}
		this.transitionTo("incomplete_expired");
	}

	cancel(reason: string | null, feedback: string | null): void {
		if (this.status === "canceled") {
			throw new BusinessError(
				"A assinatura já está cancelada.",
				"SUBSCRIPTION_ALREADY_CANCELED",
			);
		}
		this.transitionTo("canceled");
		this.canceledAt = new Date();
		this.cancelReason = reason;
		this.cancelFeedback = feedback;
		this.endsAt = this.currentPeriodEnd;
	}

	scheduleCancellation(reason: string | null, feedback: string | null): void {
		if (this.status !== "active" && this.status !== "trialing") {
			throw new BusinessError(
				"Somente assinaturas ativas ou em trial podem agendar cancelamento.",
				"SUBSCRIPTION_CANNOT_SCHEDULE_CANCELLATION",
			);
		}
		this.cancelAtPeriodEnd = true;
		this.cancelReason = reason;
		this.cancelFeedback = feedback;
		this.updatedAt = new Date();
	}

	undoCancellationSchedule(): void {
		if (!this.cancelAtPeriodEnd) {
			throw new BusinessError(
				"Não há cancelamento agendado para desfazer.",
				"NO_SCHEDULED_CANCELLATION",
			);
		}
		this.cancelAtPeriodEnd = false;
		this.cancelReason = null;
		this.cancelFeedback = null;
		this.updatedAt = new Date();
	}

	pause(reason: string | null, resumesAt: Date | null): void {
		if (this.status !== "active") {
			throw new BusinessError(
				"Somente assinaturas ativas podem ser pausadas.",
				"SUBSCRIPTION_NOT_ACTIVE",
			);
		}
		this.transitionTo("paused");
		this.pausedAt = new Date();
		this.resumesAt = resumesAt;
		this.pauseReason = reason;
	}

	resume(): void {
		if (this.status !== "paused") {
			throw new BusinessError(
				"Somente assinaturas pausadas podem ser retomadas.",
				"SUBSCRIPTION_NOT_PAUSED",
			);
		}
		this.transitionTo("active");
		this.pausedAt = null;
		this.resumesAt = null;
		this.pauseReason = null;
	}

	renewPeriod(newStart: Date, newEnd: Date, nextBilling: Date | null): void {
		if (this.status !== "active") {
			throw new BusinessError(
				"Somente assinaturas ativas podem renovar período.",
				"SUBSCRIPTION_NOT_ACTIVE",
			);
		}
		this.currentPeriodStart = newStart;
		this.currentPeriodEnd = newEnd;
		this.nextBillingDate = nextBilling;
		this.updatedAt = new Date();
	}

	changePlan(
		newPlanId: string,
		newAmount: Money | null,
		newBillingCycle: BillingCycle | null,
		newProviderPriceId: string | null,
	): void {
		if (this.status !== "active" && this.status !== "trialing") {
			throw new BusinessError(
				"Somente assinaturas ativas ou em trial podem trocar de plano.",
				"SUBSCRIPTION_CANNOT_CHANGE_PLAN",
			);
		}
		this.planId = newPlanId;
		this.amount = newAmount;
		this.billingCycle = newBillingCycle;
		this.providerPriceId = newProviderPriceId;
		this.updatedAt = new Date();
	}

	updatePaymentMethod(paymentMethodId: string | null): void {
		this.defaultPaymentMethodId = paymentMethodId;
		this.updatedAt = new Date();
	}

	updateProviderIds(
		subscriptionId: string | null,
		customerId: string | null,
		priceId: string | null,
	): void {
		this.providerSubscriptionId = subscriptionId;
		this.providerCustomerId = customerId;
		this.providerPriceId = priceId;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			id: this.id,
			userId: this.userId,
			planId: this.planId,
			status: this.status,
			billingCycle: this.billingCycle,
			amount: this.amount?.toCents() ?? null,
			currency: this.currency,
			currentPeriodStart: this.currentPeriodStart,
			currentPeriodEnd: this.currentPeriodEnd,
			trialStart: this.trialStart,
			trialEnd: this.trialEnd,
			startsAt: this.startsAt,
			endsAt: this.endsAt,
			nextBillingDate: this.nextBillingDate,
			cancelAtPeriodEnd: this.cancelAtPeriodEnd,
			canceledAt: this.canceledAt,
			cancelReason: this.cancelReason,
			cancelFeedback: this.cancelFeedback,
			pausedAt: this.pausedAt,
			resumesAt: this.resumesAt,
			pauseReason: this.pauseReason,
			defaultPaymentMethodId: this.defaultPaymentMethodId,
			collectionMethod: this.collectionMethod,
			daysUntilDue: this.daysUntilDue,
			latestInvoiceId: this.latestInvoiceId,
			pendingUpdateId: this.pendingUpdateId,
			provider: this.provider,
			providerSubscriptionId: this.providerSubscriptionId,
			providerCustomerId: this.providerCustomerId,
			providerPriceId: this.providerPriceId,
			metadata: this.metadata,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}

	static create(props: SubscriptionProps): Subscription {
		return new Subscription(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		planId: string;
		status: SubscriptionStatus;
		billingCycle: BillingCycle | null;
		amount: number | null;
		currency: string;
		currentPeriodStart: Date;
		currentPeriodEnd: Date;
		trialStart: Date | null;
		trialEnd: Date | null;
		startsAt: Date;
		endsAt: Date | null;
		nextBillingDate: Date | null;
		cancelAtPeriodEnd: boolean | null;
		canceledAt: Date | null;
		cancelReason: string | null;
		cancelFeedback: string | null;
		pausedAt: Date | null;
		resumesAt: Date | null;
		pauseReason: string | null;
		defaultPaymentMethodId: string | null;
		collectionMethod: CollectionMethod | null;
		daysUntilDue: number | null;
		latestInvoiceId: string | null;
		pendingUpdateId: string | null;
		provider: string;
		providerSubscriptionId: string | null;
		providerCustomerId: string | null;
		providerPriceId: string | null;
		metadata: Record<string, unknown> | null;
		createdAt: Date;
		updatedAt: Date;
	}): Subscription {
		const { Money } = require("../value-objects/money.value-object");

		return new Subscription({
			id: data.id,
			userId: data.userId,
			planId: data.planId,
			status: data.status,
			billingCycle: data.billingCycle,
			amount: data.amount !== null ? Money.fromCents(data.amount, data.currency) : null,
			currency: data.currency,
			currentPeriodStart: data.currentPeriodStart,
			currentPeriodEnd: data.currentPeriodEnd,
			trialStart: data.trialStart,
			trialEnd: data.trialEnd,
			startsAt: data.startsAt,
			endsAt: data.endsAt,
			nextBillingDate: data.nextBillingDate,
			cancelAtPeriodEnd: data.cancelAtPeriodEnd ?? false,
			canceledAt: data.canceledAt,
			cancelReason: data.cancelReason,
			cancelFeedback: data.cancelFeedback,
			pausedAt: data.pausedAt,
			resumesAt: data.resumesAt,
			pauseReason: data.pauseReason,
			defaultPaymentMethodId: data.defaultPaymentMethodId,
			collectionMethod: data.collectionMethod,
			daysUntilDue: data.daysUntilDue,
			latestInvoiceId: data.latestInvoiceId,
			pendingUpdateId: data.pendingUpdateId,
			provider: data.provider,
			providerSubscriptionId: data.providerSubscriptionId,
			providerCustomerId: data.providerCustomerId,
			providerPriceId: data.providerPriceId,
			metadata: data.metadata,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/billing/domain/entities/payment-method.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { CardBrand, PaymentMethodStatus, PaymentMethodType } from "../enums";

interface CardDetails {
	brand: CardBrand | null;
	last4: string | null;
	expiryMonth: number | null;
	expiryYear: number | null;
	fingerprint: string | null;
	funding: string | null;
	country: string | null;
}

interface PixDetails {
	key: string | null;
	keyType: string | null;
}

interface BoletoDetails {
	taxId: string | null;
}

interface BankDetails {
	name: string | null;
	code: string | null;
	accountLast4: string | null;
	accountType: string | null;
	branchCode: string | null;
}

interface PaymentMethodProps {
	id: string;
	userId: string;
	type: PaymentMethodType;
	status: PaymentMethodStatus;
	isDefault: boolean;
	label: string | null;
	provider: string;
	providerPaymentMethodId: string | null;
	providerCustomerId: string | null;
	cardDetails: CardDetails | null;
	pixDetails: PixDetails | null;
	boletoDetails: BoletoDetails | null;
	bankDetails: BankDetails | null;
	supportsRecurring: boolean;
	supportsInstallments: boolean;
	requiresAuthentication: boolean;
	maxInstallments: number | null;
	metadata: Record<string, unknown> | null;
	lastUsedAt: Date | null;
	verifiedAt: Date | null;
	expiresAt: Date | null;
	createdAt: Date;
	updatedAt: Date;
}

export class PaymentMethod {
	private readonly id: string;
	private userId: string;
	private type: PaymentMethodType;
	private status: PaymentMethodStatus;
	private isDefault: boolean;
	private label: string | null;
	private provider: string;
	private providerPaymentMethodId: string | null;
	private providerCustomerId: string | null;
	private cardDetails: CardDetails | null;
	private pixDetails: PixDetails | null;
	private boletoDetails: BoletoDetails | null;
	private bankDetails: BankDetails | null;
	private supportsRecurring: boolean;
	private supportsInstallments: boolean;
	private requiresAuthentication: boolean;
	private maxInstallments: number | null;
	private metadata: Record<string, unknown> | null;
	private lastUsedAt: Date | null;
	private verifiedAt: Date | null;
	private expiresAt: Date | null;
	private readonly createdAt: Date;
	private updatedAt: Date;

	constructor(props: PaymentMethodProps) {
		this.id = props.id;
		this.userId = props.userId;
		this.type = props.type;
		this.status = props.status;
		this.isDefault = props.isDefault;
		this.label = props.label;
		this.provider = props.provider;
		this.providerPaymentMethodId = props.providerPaymentMethodId;
		this.providerCustomerId = props.providerCustomerId;
		this.cardDetails = props.cardDetails;
		this.pixDetails = props.pixDetails;
		this.boletoDetails = props.boletoDetails;
		this.bankDetails = props.bankDetails;
		this.supportsRecurring = props.supportsRecurring;
		this.supportsInstallments = props.supportsInstallments;
		this.requiresAuthentication = props.requiresAuthentication;
		this.maxInstallments = props.maxInstallments;
		this.metadata = props.metadata;
		this.lastUsedAt = props.lastUsedAt;
		this.verifiedAt = props.verifiedAt;
		this.expiresAt = props.expiresAt;
		this.createdAt = props.createdAt;
		this.updatedAt = props.updatedAt;
	}

	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getType(): PaymentMethodType {
		return this.type;
	}

	getStatus(): PaymentMethodStatus {
		return this.status;
	}

	getIsDefault(): boolean {
		return this.isDefault;
	}

	getLabel(): string | null {
		return this.label;
	}

	getProvider(): string {
		return this.provider;
	}

	getProviderPaymentMethodId(): string | null {
		return this.providerPaymentMethodId;
	}

	getProviderCustomerId(): string | null {
		return this.providerCustomerId;
	}

	getCardDetails(): CardDetails | null {
		return this.cardDetails ? { ...this.cardDetails } : null;
	}

	getPixDetails(): PixDetails | null {
		return this.pixDetails ? { ...this.pixDetails } : null;
	}

	getBoletoDetails(): BoletoDetails | null {
		return this.boletoDetails ? { ...this.boletoDetails } : null;
	}

	getBankDetails(): BankDetails | null {
		return this.bankDetails ? { ...this.bankDetails } : null;
	}

	getSupportsRecurring(): boolean {
		return this.supportsRecurring;
	}

	getSupportsInstallments(): boolean {
		return this.supportsInstallments;
	}

	getRequiresAuthentication(): boolean {
		return this.requiresAuthentication;
	}

	getMaxInstallments(): number | null {
		return this.maxInstallments;
	}

	getMetadata(): Record<string, unknown> | null {
		return this.metadata ? { ...this.metadata } : null;
	}

	getLastUsedAt(): Date | null {
		return this.lastUsedAt;
	}

	getVerifiedAt(): Date | null {
		return this.verifiedAt;
	}

	getExpiresAt(): Date | null {
		return this.expiresAt;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	isCard(): boolean {
		return this.type === "card";
	}

	isPix(): boolean {
		return this.type === "pix";
	}

	isBoleto(): boolean {
		return this.type === "boleto";
	}

	isWallet(): boolean {
		return this.type === "google_pay" || this.type === "apple_pay";
	}

	canBeUsedForRecurring(): boolean {
		return this.supportsRecurring && this.status === "active";
	}

	isExpired(): boolean {
		if (this.cardDetails?.expiryMonth && this.cardDetails?.expiryYear) {
			const now = new Date();
			const expiryDate = new Date(
				this.cardDetails.expiryYear,
				this.cardDetails.expiryMonth,
				0,
			);
			return now > expiryDate;
		}
		if (this.expiresAt) {
			return new Date() > this.expiresAt;
		}
		return false;
	}

	isUsable(): boolean {
		return this.status === "active" && !this.isExpired();
	}

	activate(): void {
		if (this.status === "active") {
			throw new BusinessError(
				"O método de pagamento já está ativo.",
				"PAYMENT_METHOD_ALREADY_ACTIVE",
			);
		}
		this.status = "active";
		this.updatedAt = new Date();
	}

	deactivate(): void {
		if (this.status === "inactive") {
			throw new BusinessError(
				"O método de pagamento já está inativo.",
				"PAYMENT_METHOD_ALREADY_INACTIVE",
			);
		}
		this.status = "inactive";
		this.updatedAt = new Date();
	}

	markExpired(): void {
		this.status = "expired";
		this.updatedAt = new Date();
	}

	markRequiresAction(): void {
		this.status = "requires_action";
		this.updatedAt = new Date();
	}

	markFailed(): void {
		this.status = "failed";
		this.updatedAt = new Date();
	}

	setAsDefault(): void {
		if (this.status !== "active") {
			throw new BusinessError(
				"Somente métodos de pagamento ativos podem ser definidos como padrão.",
				"PAYMENT_METHOD_NOT_ACTIVE",
			);
		}
		this.isDefault = true;
		this.updatedAt = new Date();
	}

	removeDefault(): void {
		this.isDefault = false;
		this.updatedAt = new Date();
	}

	recordUsage(): void {
		this.lastUsedAt = new Date();
		this.updatedAt = new Date();
	}

	updateLabel(label: string | null): void {
		this.label = label;
		this.updatedAt = new Date();
	}

	updateCardDetails(details: Partial<CardDetails>): void {
		if (!this.isCard()) {
			throw new BusinessError(
				"Somente métodos do tipo cartão podem ter dados de cartão atualizados.",
				"PAYMENT_METHOD_NOT_CARD",
			);
		}
		this.cardDetails = { ...(this.cardDetails ?? { brand: null, last4: null, expiryMonth: null, expiryYear: null, fingerprint: null, funding: null, country: null }), ...details };
		this.updatedAt = new Date();
	}

	updateProviderIds(
		paymentMethodId: string | null,
		customerId: string | null,
	): void {
		this.providerPaymentMethodId = paymentMethodId;
		this.providerCustomerId = customerId;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			id: this.id,
			userId: this.userId,
			type: this.type,
			status: this.status,
			isDefault: this.isDefault,
			label: this.label,
			provider: this.provider,
			providerPaymentMethodId: this.providerPaymentMethodId,
			providerCustomerId: this.providerCustomerId,
			cardBrand: this.cardDetails?.brand ?? null,
			cardLast4: this.cardDetails?.last4 ?? null,
			cardExpiryMonth: this.cardDetails?.expiryMonth ?? null,
			cardExpiryYear: this.cardDetails?.expiryYear ?? null,
			cardFingerprint: this.cardDetails?.fingerprint ?? null,
			cardFunding: this.cardDetails?.funding ?? null,
			cardCountry: this.cardDetails?.country ?? null,
			pixKey: this.pixDetails?.key ?? null,
			pixKeyType: this.pixDetails?.keyType ?? null,
			boletoTaxId: this.boletoDetails?.taxId ?? null,
			bankName: this.bankDetails?.name ?? null,
			bankCode: this.bankDetails?.code ?? null,
			bankAccountLast4: this.bankDetails?.accountLast4 ?? null,
			bankAccountType: this.bankDetails?.accountType ?? null,
			bankBranchCode: this.bankDetails?.branchCode ?? null,
			supportsRecurring: this.supportsRecurring,
			supportsInstallments: this.supportsInstallments,
			requiresAuthentication: this.requiresAuthentication,
			maxInstallments: this.maxInstallments,
			metadata: this.metadata,
			lastUsedAt: this.lastUsedAt,
			verifiedAt: this.verifiedAt,
			expiresAt: this.expiresAt,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}

	static create(props: PaymentMethodProps): PaymentMethod {
		return new PaymentMethod(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		type: PaymentMethodType;
		status: PaymentMethodStatus;
		isDefault: boolean;
		label: string | null;
		provider: string;
		providerPaymentMethodId: string | null;
		providerCustomerId: string | null;
		cardBrand: CardBrand | null;
		cardLast4: string | null;
		cardExpiryMonth: number | null;
		cardExpiryYear: number | null;
		cardFingerprint: string | null;
		cardFunding: string | null;
		cardCountry: string | null;
		pixKey: string | null;
		pixKeyType: string | null;
		boletoTaxId: string | null;
		bankName: string | null;
		bankCode: string | null;
		bankAccountLast4: string | null;
		bankAccountType: string | null;
		bankBranchCode: string | null;
		supportsRecurring: boolean;
		supportsInstallments: boolean;
		requiresAuthentication: boolean;
		maxInstallments: number | null;
		metadata: Record<string, unknown> | null;
		lastUsedAt: Date | null;
		verifiedAt: Date | null;
		expiresAt: Date | null;
		createdAt: Date;
		updatedAt: Date;
	}): PaymentMethod {
		const cardDetails: CardDetails | null =
			data.type === "card"
				? {
						brand: data.cardBrand,
						last4: data.cardLast4,
						expiryMonth: data.cardExpiryMonth,
						expiryYear: data.cardExpiryYear,
						fingerprint: data.cardFingerprint,
						funding: data.cardFunding,
						country: data.cardCountry,
					}
				: null;

		const pixDetails: PixDetails | null =
			data.type === "pix"
				? {
						key: data.pixKey,
						keyType: data.pixKeyType,
					}
				: null;

		const boletoDetails: BoletoDetails | null =
			data.type === "boleto"
				? {
						taxId: data.boletoTaxId,
					}
				: null;

		const bankDetails: BankDetails | null =
			data.type === "bank_transfer"
				? {
						name: data.bankName,
						code: data.bankCode,
						accountLast4: data.bankAccountLast4,
						accountType: data.bankAccountType,
						branchCode: data.bankBranchCode,
					}
				: null;

		return new PaymentMethod({
			id: data.id,
			userId: data.userId,
			type: data.type,
			status: data.status,
			isDefault: data.isDefault,
			label: data.label,
			provider: data.provider,
			providerPaymentMethodId: data.providerPaymentMethodId,
			providerCustomerId: data.providerCustomerId,
			cardDetails,
			pixDetails,
			boletoDetails,
			bankDetails,
			supportsRecurring: data.supportsRecurring,
			supportsInstallments: data.supportsInstallments,
			requiresAuthentication: data.requiresAuthentication,
			maxInstallments: data.maxInstallments,
			metadata: data.metadata,
			lastUsedAt: data.lastUsedAt,
			verifiedAt: data.verifiedAt,
			expiresAt: data.expiresAt,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/billing/domain/entities/transaction.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type {
	PaymentMethodType,
	TransactionStatus,
	TransactionType,
} from "../enums";
import { Money } from "../value-objects/money.value-object";

interface TransactionProps {
	id: string;
	userId: string;
	subscriptionId: string | null;
	paymentMethodId: string | null;
	originalTransactionId: string | null;
	type: TransactionType;
	status: TransactionStatus;
	amount: Money;
	amountGross: Money | null;
	amountFees: Money | null;
	amountNet: Money | null;
	amountRefunded: Money;
	currency: string;
	paymentMethod: PaymentMethodType | null;
	description: string | null;
	statementDescriptor: string | null;
	receiptNumber: string | null;
	receiptUrl: string | null;
	provider: string;
	providerTransactionId: string | null;
	providerPaymentIntentId: string | null;
	providerChargeId: string | null;
	providerBalanceTransactionId: string | null;
	providerInvoiceId: string | null;
	providerRefundId: string | null;
	providerDisputeId: string | null;
	errorCode: string | null;
	errorMessage: string | null;
	errorDeclineCode: string | null;
	errorType: string | null;
	disputeReason: string | null;
	disputeStatus: string | null;
	disputeAmount: Money | null;
	refundReason: string | null;
	refundedBy: string | null;
	metadata: Record<string, unknown> | null;
	internalNotes: string | null;
	ipAddress: string | null;
	userAgent: string | null;
	country: string | null;
	processedAt: Date | null;
	completedAt: Date | null;
	failedAt: Date | null;
	refundedAt: Date | null;
	disputedAt: Date | null;
	createdAt: Date;
	updatedAt: Date;
}

export class Transaction {
	private readonly id: string;
	private readonly userId: string;
	private readonly subscriptionId: string | null;
	private readonly paymentMethodId: string | null;
	private readonly originalTransactionId: string | null;
	private readonly type: TransactionType;
	private status: TransactionStatus;
	private readonly amount: Money;
	private readonly amountGross: Money | null;
	private readonly amountFees: Money | null;
	private readonly amountNet: Money | null;
	private amountRefunded: Money;
	private readonly currency: string;
	private readonly paymentMethod: PaymentMethodType | null;
	private readonly description: string | null;
	private readonly statementDescriptor: string | null;
	private receiptNumber: string | null;
	private receiptUrl: string | null;
	private readonly provider: string;
	private providerTransactionId: string | null;
	private providerPaymentIntentId: string | null;
	private providerChargeId: string | null;
	private providerBalanceTransactionId: string | null;
	private providerInvoiceId: string | null;
	private providerRefundId: string | null;
	private providerDisputeId: string | null;
	private errorCode: string | null;
	private errorMessage: string | null;
	private errorDeclineCode: string | null;
	private errorType: string | null;
	private disputeReason: string | null;
	private disputeStatus: string | null;
	private disputeAmount: Money | null;
	private refundReason: string | null;
	private refundedBy: string | null;
	private metadata: Record<string, unknown> | null;
	private internalNotes: string | null;
	private readonly ipAddress: string | null;
	private readonly userAgent: string | null;
	private readonly country: string | null;
	private processedAt: Date | null;
	private completedAt: Date | null;
	private failedAt: Date | null;
	private refundedAt: Date | null;
	private disputedAt: Date | null;
	private readonly createdAt: Date;
	private updatedAt: Date;

	private constructor(props: TransactionProps) {
		this.id = props.id;
		this.userId = props.userId;
		this.subscriptionId = props.subscriptionId;
		this.paymentMethodId = props.paymentMethodId;
		this.originalTransactionId = props.originalTransactionId;
		this.type = props.type;
		this.status = props.status;
		this.amount = props.amount;
		this.amountGross = props.amountGross;
		this.amountFees = props.amountFees;
		this.amountNet = props.amountNet;
		this.amountRefunded = props.amountRefunded;
		this.currency = props.currency;
		this.paymentMethod = props.paymentMethod;
		this.description = props.description;
		this.statementDescriptor = props.statementDescriptor;
		this.receiptNumber = props.receiptNumber;
		this.receiptUrl = props.receiptUrl;
		this.provider = props.provider;
		this.providerTransactionId = props.providerTransactionId;
		this.providerPaymentIntentId = props.providerPaymentIntentId;
		this.providerChargeId = props.providerChargeId;
		this.providerBalanceTransactionId = props.providerBalanceTransactionId;
		this.providerInvoiceId = props.providerInvoiceId;
		this.providerRefundId = props.providerRefundId;
		this.providerDisputeId = props.providerDisputeId;
		this.errorCode = props.errorCode;
		this.errorMessage = props.errorMessage;
		this.errorDeclineCode = props.errorDeclineCode;
		this.errorType = props.errorType;
		this.disputeReason = props.disputeReason;
		this.disputeStatus = props.disputeStatus;
		this.disputeAmount = props.disputeAmount;
		this.refundReason = props.refundReason;
		this.refundedBy = props.refundedBy;
		this.metadata = props.metadata;
		this.internalNotes = props.internalNotes;
		this.ipAddress = props.ipAddress;
		this.userAgent = props.userAgent;
		this.country = props.country;
		this.processedAt = props.processedAt;
		this.completedAt = props.completedAt;
		this.failedAt = props.failedAt;
		this.refundedAt = props.refundedAt;
		this.disputedAt = props.disputedAt;
		this.createdAt = props.createdAt;
		this.updatedAt = props.updatedAt;
	}

	// Getters

	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getSubscriptionId(): string | null {
		return this.subscriptionId;
	}

	getPaymentMethodId(): string | null {
		return this.paymentMethodId;
	}

	getOriginalTransactionId(): string | null {
		return this.originalTransactionId;
	}

	getType(): TransactionType {
		return this.type;
	}

	getStatus(): TransactionStatus {
		return this.status;
	}

	getAmount(): Money {
		return this.amount;
	}

	getAmountGross(): Money | null {
		return this.amountGross;
	}

	getAmountFees(): Money | null {
		return this.amountFees;
	}

	getAmountNet(): Money | null {
		return this.amountNet;
	}

	getAmountRefunded(): Money {
		return this.amountRefunded;
	}

	getCurrency(): string {
		return this.currency;
	}

	getPaymentMethod(): PaymentMethodType | null {
		return this.paymentMethod;
	}

	getDescription(): string | null {
		return this.description;
	}

	getStatementDescriptor(): string | null {
		return this.statementDescriptor;
	}

	getReceiptNumber(): string | null {
		return this.receiptNumber;
	}

	getReceiptUrl(): string | null {
		return this.receiptUrl;
	}

	getProvider(): string {
		return this.provider;
	}

	getProviderTransactionId(): string | null {
		return this.providerTransactionId;
	}

	getProviderPaymentIntentId(): string | null {
		return this.providerPaymentIntentId;
	}

	getProviderChargeId(): string | null {
		return this.providerChargeId;
	}

	getProviderBalanceTransactionId(): string | null {
		return this.providerBalanceTransactionId;
	}

	getProviderInvoiceId(): string | null {
		return this.providerInvoiceId;
	}

	getProviderRefundId(): string | null {
		return this.providerRefundId;
	}

	getProviderDisputeId(): string | null {
		return this.providerDisputeId;
	}

	getErrorCode(): string | null {
		return this.errorCode;
	}

	getErrorMessage(): string | null {
		return this.errorMessage;
	}

	getErrorDeclineCode(): string | null {
		return this.errorDeclineCode;
	}

	getErrorType(): string | null {
		return this.errorType;
	}

	getDisputeReason(): string | null {
		return this.disputeReason;
	}

	getDisputeStatus(): string | null {
		return this.disputeStatus;
	}

	getDisputeAmount(): Money | null {
		return this.disputeAmount;
	}

	getRefundReason(): string | null {
		return this.refundReason;
	}

	getRefundedBy(): string | null {
		return this.refundedBy;
	}

	getMetadata(): Record<string, unknown> | null {
		return this.metadata ? { ...this.metadata } : null;
	}

	getInternalNotes(): string | null {
		return this.internalNotes;
	}

	getIpAddress(): string | null {
		return this.ipAddress;
	}

	getUserAgent(): string | null {
		return this.userAgent;
	}

	getCountry(): string | null {
		return this.country;
	}

	getProcessedAt(): Date | null {
		return this.processedAt;
	}

	getCompletedAt(): Date | null {
		return this.completedAt;
	}

	getFailedAt(): Date | null {
		return this.failedAt;
	}

	getRefundedAt(): Date | null {
		return this.refundedAt;
	}

	getDisputedAt(): Date | null {
		return this.disputedAt;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	// Queries

	isSucceeded(): boolean {
		return this.status === "succeeded";
	}

	isFailed(): boolean {
		return this.status === "failed";
	}

	isPending(): boolean {
		return this.status === "pending";
	}

	isRefunded(): boolean {
		return this.status === "refunded";
	}

	isPartiallyRefunded(): boolean {
		return this.status === "partially_refunded";
	}

	isDisputed(): boolean {
		return this.status === "disputed";
	}

	canBeRefunded(): boolean {
		return this.status === "succeeded" || this.status === "partially_refunded";
	}

	getRemainingRefundable(): Money {
		return this.amount.subtractSafe(this.amountRefunded);
	}

	// Commands

	markProcessing(): void {
		if (this.status !== "pending") {
			throw new BusinessError(
				"Somente transações pendentes podem ser marcadas como em processamento.",
				"TRANSACTION_NOT_PENDING",
			);
		}
		this.status = "processing";
		this.processedAt = new Date();
		this.updatedAt = new Date();
	}

	markRequiresAction(): void {
		if (this.status !== "pending" && this.status !== "processing") {
			throw new BusinessError(
				"Somente transações pendentes ou em processamento podem requerer ação.",
				"TRANSACTION_INVALID_STATUS_FOR_REQUIRES_ACTION",
			);
		}
		this.status = "requires_action";
		this.updatedAt = new Date();
	}

	complete(): void {
		if (
			this.status !== "pending" &&
			this.status !== "processing" &&
			this.status !== "requires_action"
		) {
			throw new BusinessError(
				"Somente transações pendentes, em processamento ou que requerem ação podem ser completadas.",
				"TRANSACTION_CANNOT_COMPLETE",
			);
		}
		this.status = "succeeded";
		this.completedAt = new Date();
		this.updatedAt = new Date();
	}

	fail(
		errorCode: string | null,
		errorMessage: string | null,
		errorDeclineCode: string | null,
		errorType: string | null,
	): void {
		if (
			this.status !== "pending" &&
			this.status !== "processing" &&
			this.status !== "requires_action"
		) {
			throw new BusinessError(
				"Somente transações pendentes, em processamento ou que requerem ação podem falhar.",
				"TRANSACTION_CANNOT_FAIL",
			);
		}
		this.status = "failed";
		this.errorCode = errorCode;
		this.errorMessage = errorMessage;
		this.errorDeclineCode = errorDeclineCode;
		this.errorType = errorType;
		this.failedAt = new Date();
		this.updatedAt = new Date();
	}

	markRefunded(
		refundAmount: Money,
		reason: string | null,
		refundedBy: string | null,
		providerRefundId: string | null,
	): void {
		if (!this.canBeRefunded()) {
			throw new BusinessError(
				"Esta transação não pode ser reembolsada.",
				"TRANSACTION_CANNOT_REFUND",
			);
		}

		const newAmountRefunded = this.amountRefunded.add(refundAmount);

		if (newAmountRefunded.greaterThan(this.amount)) {
			throw new BusinessError(
				"O valor total de reembolso não pode exceder o valor da transação.",
				"REFUND_EXCEEDS_TRANSACTION_AMOUNT",
			);
		}

		this.amountRefunded = newAmountRefunded;
		this.status = newAmountRefunded.equals(this.amount)
			? "refunded"
			: "partially_refunded";
		this.refundReason = reason;
		this.refundedBy = refundedBy;
		this.providerRefundId = providerRefundId;
		this.refundedAt = new Date();
		this.updatedAt = new Date();
	}

	markDisputed(
		reason: string | null,
		disputeAmount: Money,
		providerDisputeId: string | null,
	): void {
		if (this.status !== "succeeded") {
			throw new BusinessError(
				"Somente transações bem-sucedidas podem ser disputadas.",
				"TRANSACTION_CANNOT_DISPUTE",
			);
		}
		this.status = "disputed";
		this.disputeReason = reason;
		this.disputeStatus = "open";
		this.disputeAmount = disputeAmount;
		this.providerDisputeId = providerDisputeId;
		this.disputedAt = new Date();
		this.updatedAt = new Date();
	}

	voidTransaction(): void {
		if (
			this.status !== "pending" &&
			this.status !== "processing" &&
			this.status !== "requires_action"
		) {
			throw new BusinessError(
				"Somente transações pendentes, em processamento ou que requerem ação podem ser anuladas.",
				"TRANSACTION_CANNOT_VOID",
			);
		}
		this.status = "void";
		this.updatedAt = new Date();
	}

	cancel(): void {
		if (
			this.status !== "pending" &&
			this.status !== "processing" &&
			this.status !== "requires_action"
		) {
			throw new BusinessError(
				"Somente transações pendentes, em processamento ou que requerem ação podem ser canceladas.",
				"TRANSACTION_CANNOT_CANCEL",
			);
		}
		this.status = "canceled";
		this.updatedAt = new Date();
	}

	updateProviderIds(
		transactionId: string | null,
		paymentIntentId: string | null,
		chargeId: string | null,
		balanceTransactionId: string | null,
		invoiceId: string | null,
	): void {
		this.providerTransactionId = transactionId;
		this.providerPaymentIntentId = paymentIntentId;
		this.providerChargeId = chargeId;
		this.providerBalanceTransactionId = balanceTransactionId;
		this.providerInvoiceId = invoiceId;
		this.updatedAt = new Date();
	}

	addInternalNote(note: string): void {
		this.internalNotes = this.internalNotes
			? `${this.internalNotes}\n${note}`
			: note;
		this.updatedAt = new Date();
	}

	toDto() {
		return {
			id: this.id,
			userId: this.userId,
			subscriptionId: this.subscriptionId,
			paymentMethodId: this.paymentMethodId,
			originalTransactionId: this.originalTransactionId,
			type: this.type,
			status: this.status,
			amount: this.amount.toCents(),
			amountGross: this.amountGross?.toCents() ?? null,
			amountFees: this.amountFees?.toCents() ?? null,
			amountNet: this.amountNet?.toCents() ?? null,
			amountRefunded: this.amountRefunded.toCents(),
			currency: this.currency,
			paymentMethod: this.paymentMethod,
			description: this.description,
			statementDescriptor: this.statementDescriptor,
			receiptNumber: this.receiptNumber,
			receiptUrl: this.receiptUrl,
			provider: this.provider,
			providerTransactionId: this.providerTransactionId,
			providerPaymentIntentId: this.providerPaymentIntentId,
			providerChargeId: this.providerChargeId,
			providerBalanceTransactionId: this.providerBalanceTransactionId,
			providerInvoiceId: this.providerInvoiceId,
			providerRefundId: this.providerRefundId,
			providerDisputeId: this.providerDisputeId,
			errorCode: this.errorCode,
			errorMessage: this.errorMessage,
			errorDeclineCode: this.errorDeclineCode,
			errorType: this.errorType,
			disputeReason: this.disputeReason,
			disputeStatus: this.disputeStatus,
			disputeAmount: this.disputeAmount?.toCents() ?? null,
			refundReason: this.refundReason,
			refundedBy: this.refundedBy,
			metadata: this.metadata,
			internalNotes: this.internalNotes,
			ipAddress: this.ipAddress,
			userAgent: this.userAgent,
			country: this.country,
			processedAt: this.processedAt,
			completedAt: this.completedAt,
			failedAt: this.failedAt,
			refundedAt: this.refundedAt,
			disputedAt: this.disputedAt,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}

	static create(props: TransactionProps): Transaction {
		return new Transaction(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		subscriptionId: string | null;
		paymentMethodId: string | null;
		originalTransactionId: string | null;
		type: TransactionType;
		status: TransactionStatus;
		amount: number;
		amountGross: number | null;
		amountFees: number | null;
		amountNet: number | null;
		amountRefunded: number;
		currency: string;
		paymentMethod: PaymentMethodType | null;
		description: string | null;
		statementDescriptor: string | null;
		receiptNumber: string | null;
		receiptUrl: string | null;
		provider: string;
		providerTransactionId: string | null;
		providerPaymentIntentId: string | null;
		providerChargeId: string | null;
		providerBalanceTransactionId: string | null;
		providerInvoiceId: string | null;
		providerRefundId: string | null;
		providerDisputeId: string | null;
		errorCode: string | null;
		errorMessage: string | null;
		errorDeclineCode: string | null;
		errorType: string | null;
		disputeReason: string | null;
		disputeStatus: string | null;
		disputeAmount: number | null;
		refundReason: string | null;
		refundedBy: string | null;
		metadata: Record<string, unknown> | null;
		internalNotes: string | null;
		ipAddress: string | null;
		userAgent: string | null;
		country: string | null;
		processedAt: Date | null;
		completedAt: Date | null;
		failedAt: Date | null;
		refundedAt: Date | null;
		disputedAt: Date | null;
		createdAt: Date;
		updatedAt: Date;
	}): Transaction {
		return new Transaction({
			id: data.id,
			userId: data.userId,
			subscriptionId: data.subscriptionId,
			paymentMethodId: data.paymentMethodId,
			originalTransactionId: data.originalTransactionId,
			type: data.type,
			status: data.status,
			amount: Money.fromCents(data.amount, data.currency),
			amountGross:
				data.amountGross !== null
					? Money.fromCents(data.amountGross, data.currency)
					: null,
			amountFees:
				data.amountFees !== null
					? Money.fromCents(data.amountFees, data.currency)
					: null,
			amountNet:
				data.amountNet !== null
					? Money.fromCents(data.amountNet, data.currency)
					: null,
			amountRefunded: Money.fromCents(data.amountRefunded, data.currency),
			currency: data.currency,
			paymentMethod: data.paymentMethod,
			description: data.description,
			statementDescriptor: data.statementDescriptor,
			receiptNumber: data.receiptNumber,
			receiptUrl: data.receiptUrl,
			provider: data.provider,
			providerTransactionId: data.providerTransactionId,
			providerPaymentIntentId: data.providerPaymentIntentId,
			providerChargeId: data.providerChargeId,
			providerBalanceTransactionId: data.providerBalanceTransactionId,
			providerInvoiceId: data.providerInvoiceId,
			providerRefundId: data.providerRefundId,
			providerDisputeId: data.providerDisputeId,
			errorCode: data.errorCode,
			errorMessage: data.errorMessage,
			errorDeclineCode: data.errorDeclineCode,
			errorType: data.errorType,
			disputeReason: data.disputeReason,
			disputeStatus: data.disputeStatus,
			disputeAmount:
				data.disputeAmount !== null
					? Money.fromCents(data.disputeAmount, data.currency)
					: null,
			refundReason: data.refundReason,
			refundedBy: data.refundedBy,
			metadata: data.metadata,
			internalNotes: data.internalNotes,
			ipAddress: data.ipAddress,
			userAgent: data.userAgent,
			country: data.country,
			processedAt: data.processedAt,
			completedAt: data.completedAt,
			failedAt: data.failedAt,
			refundedAt: data.refundedAt,
			disputedAt: data.disputedAt,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/billing/domain/entities/invoice.entity.ts =====
import { BusinessError } from "../../../../core/errors";
import type { CollectionMethod, InvoiceStatus } from "../enums";
import { Money } from "../value-objects/money.value-object";

// ===================================
// INVOICE LINE ITEM
// ===================================

export interface InvoiceLineItem {
	description: string;
	amount: number; // em centavos
	quantity: number;
	periodStart?: Date;
	periodEnd?: Date;
	proration: boolean;
}

// ===================================
// CUSTOMER SNAPSHOT
// ===================================

interface CustomerSnapshot {
	email: string | null;
	name: string | null;
	taxId: string | null;
}

// ===================================
// INVOICE PROPS
// ===================================

interface InvoiceProps {
	id: string;
	userId: string;
	subscriptionId: string | null;
	number: string | null;
	status: InvoiceStatus;
	subtotal: Money;
	discount: Money;
	tax: Money;
	amountDue: Money;
	amountPaid: Money;
	amountRemaining: Money;
	currency: string;
	periodStart: Date | null;
	periodEnd: Date | null;
	dueDate: Date | null;
	paidAt: Date | null;
	voidedAt: Date | null;
	finalizedAt: Date | null;
	billingReason: string | null;
	collectionMethod: CollectionMethod;
	attemptCount: number;
	nextPaymentAttempt: Date | null;
	discountCode: string | null;
	discountReason: string | null;
	hostedInvoiceUrl: string | null;
	invoicePdf: string | null;
	receiptNumber: string | null;
	description: string | null;
	footer: string | null;
	memo: string | null;
	customer: CustomerSnapshot;
	provider: string;
	providerInvoiceId: string | null;
	providerCustomerId: string | null;
	providerSubscriptionId: string | null;
	providerPaymentIntentId: string | null;
	providerChargeId: string | null;
	lines: InvoiceLineItem[] | null;
	metadata: Record<string, unknown> | null;
	createdAt: Date;
	updatedAt: Date;
}

// ===================================
// VALID STATUS TRANSITIONS
// ===================================

const VALID_TRANSITIONS: Record<InvoiceStatus, InvoiceStatus[]> = {
	draft: ["open", "void"],
	open: ["paid", "void", "uncollectible"],
	paid: [], // terminal
	void: [], // terminal
	uncollectible: ["open"], // pode ser reaberta
};

export class Invoice {
	private readonly id: string;
	private readonly userId: string;
	private readonly subscriptionId: string | null;
	private number: string | null;
	private status: InvoiceStatus;
	private subtotal: Money;
	private discount: Money;
	private tax: Money;
	private amountDue: Money;
	private amountPaid: Money;
	private amountRemaining: Money;
	private readonly currency: string;
	private periodStart: Date | null;
	private periodEnd: Date | null;
	private dueDate: Date | null;
	private paidAt: Date | null;
	private voidedAt: Date | null;
	private finalizedAt: Date | null;
	private billingReason: string | null;
	private collectionMethod: CollectionMethod;
	private attemptCount: number;
	private nextPaymentAttempt: Date | null;
	private discountCode: string | null;
	private discountReason: string | null;
	private hostedInvoiceUrl: string | null;
	private invoicePdf: string | null;
	private receiptNumber: string | null;
	private description: string | null;
	private footer: string | null;
	private memo: string | null;
	private customer: CustomerSnapshot;
	private readonly provider: string;
	private providerInvoiceId: string | null;
	private providerCustomerId: string | null;
	private providerSubscriptionId: string | null;
	private providerPaymentIntentId: string | null;
	private providerChargeId: string | null;
	private lines: InvoiceLineItem[] | null;
	private metadata: Record<string, unknown> | null;
	private readonly createdAt: Date;
	private updatedAt: Date;

	private constructor(props: InvoiceProps) {
		this.id = props.id;
		this.userId = props.userId;
		this.subscriptionId = props.subscriptionId;
		this.number = props.number;
		this.status = props.status;
		this.subtotal = props.subtotal;
		this.discount = props.discount;
		this.tax = props.tax;
		this.amountDue = props.amountDue;
		this.amountPaid = props.amountPaid;
		this.amountRemaining = props.amountRemaining;
		this.currency = props.currency;
		this.periodStart = props.periodStart;
		this.periodEnd = props.periodEnd;
		this.dueDate = props.dueDate;
		this.paidAt = props.paidAt;
		this.voidedAt = props.voidedAt;
		this.finalizedAt = props.finalizedAt;
		this.billingReason = props.billingReason;
		this.collectionMethod = props.collectionMethod;
		this.attemptCount = props.attemptCount;
		this.nextPaymentAttempt = props.nextPaymentAttempt;
		this.discountCode = props.discountCode;
		this.discountReason = props.discountReason;
		this.hostedInvoiceUrl = props.hostedInvoiceUrl;
		this.invoicePdf = props.invoicePdf;
		this.receiptNumber = props.receiptNumber;
		this.description = props.description;
		this.footer = props.footer;
		this.memo = props.memo;
		this.customer = props.customer;
		this.provider = props.provider;
		this.providerInvoiceId = props.providerInvoiceId;
		this.providerCustomerId = props.providerCustomerId;
		this.providerSubscriptionId = props.providerSubscriptionId;
		this.providerPaymentIntentId = props.providerPaymentIntentId;
		this.providerChargeId = props.providerChargeId;
		this.lines = props.lines;
		this.metadata = props.metadata;
		this.createdAt = props.createdAt;
		this.updatedAt = props.updatedAt;
	}

	// ===================================
	// GETTERS
	// ===================================

	getId(): string {
		return this.id;
	}

	getUserId(): string {
		return this.userId;
	}

	getSubscriptionId(): string | null {
		return this.subscriptionId;
	}

	getNumber(): string | null {
		return this.number;
	}

	getStatus(): InvoiceStatus {
		return this.status;
	}

	getSubtotal(): Money {
		return this.subtotal;
	}

	getDiscount(): Money {
		return this.discount;
	}

	getTax(): Money {
		return this.tax;
	}

	getAmountDue(): Money {
		return this.amountDue;
	}

	getAmountPaid(): Money {
		return this.amountPaid;
	}

	getAmountRemaining(): Money {
		return this.amountRemaining;
	}

	getCurrency(): string {
		return this.currency;
	}

	getPeriodStart(): Date | null {
		return this.periodStart;
	}

	getPeriodEnd(): Date | null {
		return this.periodEnd;
	}

	getDueDate(): Date | null {
		return this.dueDate;
	}

	getPaidAt(): Date | null {
		return this.paidAt;
	}

	getVoidedAt(): Date | null {
		return this.voidedAt;
	}

	getFinalizedAt(): Date | null {
		return this.finalizedAt;
	}

	getBillingReason(): string | null {
		return this.billingReason;
	}

	getCollectionMethod(): CollectionMethod {
		return this.collectionMethod;
	}

	getAttemptCount(): number {
		return this.attemptCount;
	}

	getNextPaymentAttempt(): Date | null {
		return this.nextPaymentAttempt;
	}

	getDiscountCode(): string | null {
		return this.discountCode;
	}

	getDiscountReason(): string | null {
		return this.discountReason;
	}

	getHostedInvoiceUrl(): string | null {
		return this.hostedInvoiceUrl;
	}

	getInvoicePdf(): string | null {
		return this.invoicePdf;
	}

	getReceiptNumber(): string | null {
		return this.receiptNumber;
	}

	getDescription(): string | null {
		return this.description;
	}

	getFooter(): string | null {
		return this.footer;
	}

	getMemo(): string | null {
		return this.memo;
	}

	getCustomer(): CustomerSnapshot {
		return { ...this.customer };
	}

	getProvider(): string {
		return this.provider;
	}

	getProviderInvoiceId(): string | null {
		return this.providerInvoiceId;
	}

	getProviderCustomerId(): string | null {
		return this.providerCustomerId;
	}

	getProviderSubscriptionId(): string | null {
		return this.providerSubscriptionId;
	}

	getProviderPaymentIntentId(): string | null {
		return this.providerPaymentIntentId;
	}

	getProviderChargeId(): string | null {
		return this.providerChargeId;
	}

	getLines(): InvoiceLineItem[] | null {
		return this.lines ? [...this.lines] : null;
	}

	getMetadata(): Record<string, unknown> | null {
		return this.metadata ? { ...this.metadata } : null;
	}

	getCreatedAt(): Date {
		return this.createdAt;
	}

	getUpdatedAt(): Date {
		return this.updatedAt;
	}

	// ===================================
	// QUERIES
	// ===================================

	isDraft(): boolean {
		return this.status === "draft";
	}

	isOpen(): boolean {
		return this.status === "open";
	}

	isPaid(): boolean {
		return this.status === "paid";
	}

	isVoid(): boolean {
		return this.status === "void";
	}

	isUncollectible(): boolean {
		return this.status === "uncollectible";
	}

	isEditable(): boolean {
		return this.status === "draft";
	}

	isOverdue(): boolean {
		if (this.status !== "open" || !this.dueDate) return false;
		return new Date() > this.dueDate;
	}

	isFullyPaid(): boolean {
		return this.amountRemaining.isZero();
	}

	hasDiscount(): boolean {
		return !this.discount.isZero();
	}

	// ===================================
	// COMMANDS - STATUS TRANSITIONS
	// ===================================

	private transitionTo(newStatus: InvoiceStatus): void {
		const allowed = VALID_TRANSITIONS[this.status];
		if (!allowed.includes(newStatus)) {
			throw new BusinessError(
				`Não é possível alterar o status da fatura de "${this.status}" para "${newStatus}".`,
				"INVOICE_INVALID_STATUS_TRANSITION",
			);
		}
		this.status = newStatus;
		this.updatedAt = new Date();
	}

	finalize(): void {
		if (!this.lines || this.lines.length === 0) {
			throw new BusinessError(
				"A fatura deve ter pelo menos um item para ser finalizada.",
				"INVOICE_NO_LINE_ITEMS",
			);
		}
		this.transitionTo("open");
		this.finalizedAt = new Date();
	}

	markPaid(receiptNumber: string | null): void {
		this.transitionTo("paid");
		this.amountPaid = this.amountDue;
		this.amountRemaining = Money.zero(this.currency);
		this.receiptNumber = receiptNumber;
		this.paidAt = new Date();
	}

	voidInvoice(): void {
		this.transitionTo("void");
		this.voidedAt = new Date();
	}

	markUncollectible(): void {
		this.transitionTo("uncollectible");
	}

	reopenFromUncollectible(): void {
		this.transitionTo("open");
	}

	// ===================================
	// COMMANDS - PAYMENT
	// ===================================

	recordPartialPayment(amount: Money): void {
		if (this.status !== "open") {
			throw new BusinessError(
				"Somente faturas abertas podem receber pagamentos.",
				"INVOICE_NOT_OPEN",
			);
		}

		if (amount.greaterThan(this.amountRemaining)) {
			throw new BusinessError(
				"O valor do pagamento excede o valor restante da fatura.",
				"PAYMENT_EXCEEDS_REMAINING",
			);
		}

		this.amountPaid = this.amountPaid.add(amount);
		this.amountRemaining = this.amountDue.subtractSafe(this.amountPaid);

		if (this.amountRemaining.isZero()) {
			this.status = "paid";
			this.paidAt = new Date();
		}

		this.updatedAt = new Date();
	}

	incrementAttemptCount(nextAttempt: Date | null): void {
		this.attemptCount += 1;
		this.nextPaymentAttempt = nextAttempt;
		this.updatedAt = new Date();
	}

	// ===================================
	// COMMANDS - DRAFT EDITING
	// ===================================

	updateLines(lines: InvoiceLineItem[]): void {
		if (!this.isEditable()) {
			throw new BusinessError(
				"Somente faturas em rascunho podem ser editadas.",
				"INVOICE_NOT_EDITABLE",
			);
		}
		this.lines = lines;
		this.recalculateAmounts();
		this.updatedAt = new Date();
	}

	updateDescription(description: string | null): void {
		if (!this.isEditable()) {
			throw new BusinessError(
				"Somente faturas em rascunho podem ser editadas.",
				"INVOICE_NOT_EDITABLE",
			);
		}
		this.description = description;
		this.updatedAt = new Date();
	}

	updateFooter(footer: string | null): void {
		if (!this.isEditable()) {
			throw new BusinessError(
				"Somente faturas em rascunho podem ser editadas.",
				"INVOICE_NOT_EDITABLE",
			);
		}
		this.footer = footer;
		this.updatedAt = new Date();
	}

	updateMemo(memo: string | null): void {
		this.memo = memo;
		this.updatedAt = new Date();
	}

	updateDueDate(dueDate: Date | null): void {
		if (!this.isEditable()) {
			throw new BusinessError(
				"Somente faturas em rascunho podem ser editadas.",
				"INVOICE_NOT_EDITABLE",
			);
		}
		this.dueDate = dueDate;
		this.updatedAt = new Date();
	}

	applyDiscount(
		amount: Money,
		code: string | null,
		reason: string | null,
	): void {
		if (!this.isEditable()) {
			throw new BusinessError(
				"Somente faturas em rascunho podem ser editadas.",
				"INVOICE_NOT_EDITABLE",
			);
		}
		this.discount = amount;
		this.discountCode = code;
		this.discountReason = reason;
		this.recalculateAmounts();
		this.updatedAt = new Date();
	}

	// ===================================
	// COMMANDS - PROVIDER
	// ===================================

	updateProviderIds(
		invoiceId: string | null,
		customerId: string | null,
		subscriptionId: string | null,
		paymentIntentId: string | null,
		chargeId: string | null,
	): void {
		this.providerInvoiceId = invoiceId;
		this.providerCustomerId = customerId;
		this.providerSubscriptionId = subscriptionId;
		this.providerPaymentIntentId = paymentIntentId;
		this.providerChargeId = chargeId;
		this.updatedAt = new Date();
	}

	updateDocumentUrls(
		hostedUrl: string | null,
		pdfUrl: string | null,
	): void {
		this.hostedInvoiceUrl = hostedUrl;
		this.invoicePdf = pdfUrl;
		this.updatedAt = new Date();
	}

	updateCustomerSnapshot(snapshot: CustomerSnapshot): void {
		this.customer = { ...snapshot };
		this.updatedAt = new Date();
	}

	assignNumber(invoiceNumber: string): void {
		this.number = invoiceNumber;
		this.updatedAt = new Date();
	}

	// ===================================
	// PRIVATE
	// ===================================

	private recalculateAmounts(): void {
		if (this.lines) {
			const subtotalCents = this.lines.reduce(
				(sum, line) => sum + line.amount * line.quantity,
				0,
			);
			this.subtotal = Money.fromCents(subtotalCents, this.currency);
		}

		const due = this.subtotal
			.subtractSafe(this.discount)
			.add(this.tax);
		this.amountDue = due;
		this.amountRemaining = due.subtractSafe(this.amountPaid);
	}

	// ===================================
	// SERIALIZATION
	// ===================================

	toDto() {
		return {
			id: this.id,
			userId: this.userId,
			subscriptionId: this.subscriptionId,
			number: this.number,
			status: this.status,
			subtotal: this.subtotal.toCents(),
			discount: this.discount.toCents(),
			tax: this.tax.toCents(),
			amountDue: this.amountDue.toCents(),
			amountPaid: this.amountPaid.toCents(),
			amountRemaining: this.amountRemaining.toCents(),
			currency: this.currency,
			periodStart: this.periodStart,
			periodEnd: this.periodEnd,
			dueDate: this.dueDate,
			paidAt: this.paidAt,
			voidedAt: this.voidedAt,
			finalizedAt: this.finalizedAt,
			billingReason: this.billingReason,
			collectionMethod: this.collectionMethod,
			attemptCount: this.attemptCount,
			nextPaymentAttempt: this.nextPaymentAttempt,
			discountCode: this.discountCode,
			discountReason: this.discountReason,
			hostedInvoiceUrl: this.hostedInvoiceUrl,
			invoicePdf: this.invoicePdf,
			receiptNumber: this.receiptNumber,
			description: this.description,
			footer: this.footer,
			memo: this.memo,
			customerEmail: this.customer.email,
			customerName: this.customer.name,
			customerTaxId: this.customer.taxId,
			provider: this.provider,
			providerInvoiceId: this.providerInvoiceId,
			providerCustomerId: this.providerCustomerId,
			providerSubscriptionId: this.providerSubscriptionId,
			providerPaymentIntentId: this.providerPaymentIntentId,
			providerChargeId: this.providerChargeId,
			lines: this.lines,
			metadata: this.metadata,
			createdAt: this.createdAt,
			updatedAt: this.updatedAt,
		};
	}

	// ===================================
	// FACTORIES
	// ===================================

	static create(props: InvoiceProps): Invoice {
		return new Invoice(props);
	}

	static fromPersistence(data: {
		id: string;
		userId: string;
		subscriptionId: string | null;
		number: string | null;
		status: InvoiceStatus;
		subtotal: number;
		discount: number | null;
		tax: number | null;
		amountDue: number;
		amountPaid: number | null;
		amountRemaining: number;
		currency: string;
		periodStart: Date | null;
		periodEnd: Date | null;
		dueDate: Date | null;
		paidAt: Date | null;
		voidedAt: Date | null;
		finalizedAt: Date | null;
		billingReason: string | null;
		collectionMethod: CollectionMethod;
		attemptCount: number | null;
		nextPaymentAttempt: Date | null;
		discountCode: string | null;
		discountReason: string | null;
		hostedInvoiceUrl: string | null;
		invoicePdf: string | null;
		receiptNumber: string | null;
		description: string | null;
		footer: string | null;
		memo: string | null;
		customerEmail: string | null;
		customerName: string | null;
		customerTaxId: string | null;
		provider: string;
		providerInvoiceId: string | null;
		providerCustomerId: string | null;
		providerSubscriptionId: string | null;
		providerPaymentIntentId: string | null;
		providerChargeId: string | null;
		lines: InvoiceLineItem[] | null;
		metadata: Record<string, unknown> | null;
		createdAt: Date;
		updatedAt: Date;
	}): Invoice {
		return new Invoice({
			id: data.id,
			userId: data.userId,
			subscriptionId: data.subscriptionId,
			number: data.number,
			status: data.status,
			subtotal: Money.fromCents(data.subtotal, data.currency),
			discount: Money.fromCents(data.discount ?? 0, data.currency),
			tax: Money.fromCents(data.tax ?? 0, data.currency),
			amountDue: Money.fromCents(data.amountDue, data.currency),
			amountPaid: Money.fromCents(data.amountPaid ?? 0, data.currency),
			amountRemaining: Money.fromCents(data.amountRemaining, data.currency),
			currency: data.currency,
			periodStart: data.periodStart,
			periodEnd: data.periodEnd,
			dueDate: data.dueDate,
			paidAt: data.paidAt,
			voidedAt: data.voidedAt,
			finalizedAt: data.finalizedAt,
			billingReason: data.billingReason,
			collectionMethod: data.collectionMethod,
			attemptCount: data.attemptCount ?? 0,
			nextPaymentAttempt: data.nextPaymentAttempt,
			discountCode: data.discountCode,
			discountReason: data.discountReason,
			hostedInvoiceUrl: data.hostedInvoiceUrl,
			invoicePdf: data.invoicePdf,
			receiptNumber: data.receiptNumber,
			description: data.description,
			footer: data.footer,
			memo: data.memo,
			customer: {
				email: data.customerEmail,
				name: data.customerName,
				taxId: data.customerTaxId,
			},
			provider: data.provider,
			providerInvoiceId: data.providerInvoiceId,
			providerCustomerId: data.providerCustomerId,
			providerSubscriptionId: data.providerSubscriptionId,
			providerPaymentIntentId: data.providerPaymentIntentId,
			providerChargeId: data.providerChargeId,
			lines: data.lines,
			metadata: data.metadata,
			createdAt: data.createdAt,
			updatedAt: data.updatedAt,
		});
	}
}
\n\n===== FILE: ./src/modules/billing/domain/value-objects/money.value-object.ts =====
/**
 * Money Value Object
 * Encapsula valores monetários com validação e formatação
 * Imutável - sempre cria nova instância em operações
 *
 * IMPORTANTE:
 * - Valores sempre em centavos (evita problemas de ponto flutuante)
 * - Alinhado com a Stripe que trabalha exclusivamente com centavos
 * - Suporta múltiplas moedas (ISO 4217)
 */

// Moedas válidas (ISO 4217)
const VALID_CURRENCIES = new Set([
	"BRL", // Real Brasileiro
	"USD", // Dólar Americano
	"EUR", // Euro
	"GBP", // Libra Esterlina
	"ARS", // Peso Argentino
	"CLP", // Peso Chileno
	"COP", // Peso Colombiano
	"MXN", // Peso Mexicano
	"PEN", // Sol Peruano
	"UYU", // Peso Uruguaio
]);

export class Money {
	private constructor(
		private readonly amount: number, // Em centavos
		private readonly currency: string = "BRL",
	) {
		if (amount < 0) {
			throw new Error("Amount cannot be negative");
		}
		if (!Number.isInteger(amount)) {
			throw new Error("Amount must be an integer (in cents)");
		}
		if (!currency || currency.length !== 3) {
			throw new Error("Currency must be a 3-letter ISO code (e.g., BRL, USD)");
		}
		if (!VALID_CURRENCIES.has(currency.toUpperCase())) {
			throw new Error(
				`Invalid currency: ${currency}. Supported: ${[...VALID_CURRENCIES].join(", ")}`,
			);
		}
	}

	// ===================================
	// FACTORY METHODS
	// ===================================

	/**
	 * Cria Money a partir de centavos
	 * @param cents - Valor em centavos (ex: 3990 = R$ 39,90)
	 * @param currency - Código ISO da moeda (default: "BRL")
	 */
	static fromCents(cents: number, currency: string = "BRL"): Money {
		return new Money(cents, currency.toUpperCase());
	}

	/**
	 * Cria Money a partir de unidades (reais/dólares)
	 * @param units - Valor em unidade (ex: 39.90)
	 * @param currency - Código ISO da moeda (default: "BRL")
	 */
	static fromUnits(units: number, currency: string = "BRL"): Money {
		const cents = Math.round(units * 100);
		return new Money(cents, currency.toUpperCase());
	}

	/**
	 * Cria Money zero
	 */
	static zero(currency: string = "BRL"): Money {
		return new Money(0, currency.toUpperCase());
	}

	/**
	 * Cria Money a partir de valor da Stripe
	 * @param amount - Valor em centavos (da API Stripe)
	 * @param currency - Código da moeda em lowercase (padrão Stripe)
	 */
	static fromStripeAmount(amount: number, currency: string): Money {
		return new Money(amount, currency.toUpperCase());
	}

	// ===================================
	// GETTERS
	// ===================================

	/**
	 * Retorna valor em centavos
	 */
	toCents(): number {
		return this.amount;
	}

	/**
	 * Retorna valor em unidades (reais/dólares)
	 */
	toUnits(): number {
		return this.amount / 100;
	}

	/**
	 * Retorna código da moeda
	 */
	getCurrency(): string {
		return this.currency;
	}

	// ===================================
	// STRIPE COMPATIBILITY
	// ===================================

	/**
	 * Retorna valor para API da Stripe (centavos)
	 */
	toStripeAmount(): number {
		return this.amount;
	}

	/**
	 * Retorna objeto compatível com Stripe Price/PaymentIntent
	 */
	toStripeObject(): { currency: string; unit_amount: number } {
		return {
			currency: this.currency.toLowerCase(),
			unit_amount: this.amount,
		};
	}

	// ===================================
	// FORMATTING
	// ===================================

	/**
	 * Formata para exibição
	 * Ex: R$ 39,90 ou $ 39.90
	 */
	format(locale: string = "pt-BR"): string {
		const formatter = new Intl.NumberFormat(locale, {
			style: "currency",
			currency: this.currency,
		});
		return formatter.format(this.toUnits());
	}

	/**
	 * Formata apenas o valor numérico
	 * Ex: 39,90 ou 39.90
	 */
	formatValue(locale: string = "pt-BR"): string {
		const formatter = new Intl.NumberFormat(locale, {
			minimumFractionDigits: 2,
			maximumFractionDigits: 2,
		});
		return formatter.format(this.toUnits());
	}

	// ===================================
	// ARITHMETIC OPERATIONS
	// ===================================

	/**
	 * Adiciona outro Money (mesmo currency)
	 */
	add(other: Money): Money {
		this.ensureSameCurrency(other);
		return new Money(this.amount + other.amount, this.currency);
	}

	/**
	 * Subtrai outro Money (mesmo currency)
	 */
	subtract(other: Money): Money {
		this.ensureSameCurrency(other);
		const newAmount = this.amount - other.amount;
		if (newAmount < 0) {
			throw new Error("Subtraction would result in negative amount");
		}
		return new Money(newAmount, this.currency);
	}

	/**
	 * Subtrai outro Money, retornando zero se negativo
	 */
	subtractSafe(other: Money): Money {
		this.ensureSameCurrency(other);
		const newAmount = Math.max(0, this.amount - other.amount);
		return new Money(newAmount, this.currency);
	}

	/**
	 * Multiplica por um fator
	 */
	multiply(factor: number): Money {
		if (factor < 0) {
			throw new Error("Factor cannot be negative");
		}
		return new Money(Math.round(this.amount * factor), this.currency);
	}

	/**
	 * Divide por um divisor
	 */
	divide(divisor: number): Money {
		if (divisor <= 0) {
			throw new Error("Divisor must be positive");
		}
		return new Money(Math.round(this.amount / divisor), this.currency);
	}

	/**
	 * Aplica porcentagem (ex: 10 = 10%)
	 */
	percentage(percent: number): Money {
		return new Money(Math.round(this.amount * (percent / 100)), this.currency);
	}

	/**
	 * Adiciona imposto (ex: 10 = 10% de imposto)
	 */
	withTax(taxPercent: number): Money {
		const taxAmount = Math.round(this.amount * (taxPercent / 100));
		return new Money(this.amount + taxAmount, this.currency);
	}

	/**
	 * Calcula valor do imposto (ex: 10 = 10%)
	 */
	getTaxAmount(taxPercent: number): Money {
		const taxAmount = Math.round(this.amount * (taxPercent / 100));
		return new Money(taxAmount, this.currency);
	}

	/**
	 * Aplica desconto (ex: 20 = 20% de desconto)
	 */
	withDiscount(discountPercent: number): Money {
		if (discountPercent < 0 || discountPercent > 100) {
			throw new Error("Discount percent must be between 0 and 100");
		}
		const discountAmount = Math.round(this.amount * (discountPercent / 100));
		return new Money(this.amount - discountAmount, this.currency);
	}

	/**
	 * Distribui valor em parcelas iguais
	 * Retorna array de Money com o restante na última parcela
	 */
	distribute(parts: number): Money[] {
		if (parts <= 0 || !Number.isInteger(parts)) {
			throw new Error("Parts must be a positive integer");
		}

		const baseAmount = Math.floor(this.amount / parts);
		const remainder = this.amount % parts;
		const result: Money[] = [];

		for (let i = 0; i < parts; i++) {
			const extra = i < remainder ? 1 : 0;
			result.push(new Money(baseAmount + extra, this.currency));
		}

		return result;
	}

	// ===================================
	// COMPARISON METHODS
	// ===================================

	/**
	 * Compara se é igual a outro Money
	 */
	equals(other: Money): boolean {
		return this.amount === other.amount && this.currency === other.currency;
	}

	/**
	 * Compara se é maior que outro Money
	 */
	greaterThan(other: Money): boolean {
		this.ensureSameCurrency(other);
		return this.amount > other.amount;
	}

	/**
	 * Compara se é maior ou igual a outro Money
	 */
	greaterThanOrEqual(other: Money): boolean {
		this.ensureSameCurrency(other);
		return this.amount >= other.amount;
	}

	/**
	 * Compara se é menor que outro Money
	 */
	lessThan(other: Money): boolean {
		this.ensureSameCurrency(other);
		return this.amount < other.amount;
	}

	/**
	 * Compara se é menor ou igual a outro Money
	 */
	lessThanOrEqual(other: Money): boolean {
		this.ensureSameCurrency(other);
		return this.amount <= other.amount;
	}

	/**
	 * Verifica se é zero
	 */
	isZero(): boolean {
		return this.amount === 0;
	}

	/**
	 * Verifica se é positivo (maior que zero)
	 */
	isPositive(): boolean {
		return this.amount > 0;
	}

	/**
	 * Retorna o menor valor entre dois Money
	 */
	min(other: Money): Money {
		this.ensureSameCurrency(other);
		return this.amount <= other.amount ? this : other;
	}

	/**
	 * Retorna o maior valor entre dois Money
	 */
	max(other: Money): Money {
		this.ensureSameCurrency(other);
		return this.amount >= other.amount ? this : other;
	}

	// ===================================
	// SERIALIZATION
	// ===================================

	/**
	 * Serializa para persistência (retorna centavos)
	 */
	toJSON(): { amount: number; currency: string } {
		return {
			amount: this.amount,
			currency: this.currency,
		};
	}

	/**
	 * Desserializa de persistência
	 */
	static fromJSON(json: { amount: number; currency: string }): Money {
		return new Money(json.amount, json.currency);
	}

	// ===================================
	// PRIVATE HELPERS
	// ===================================

	/**
	 * Valida se duas moedas são iguais
	 */
	private ensureSameCurrency(other: Money): void {
		if (this.currency !== other.currency) {
			throw new Error(
				`Cannot operate with different currencies: ${this.currency} and ${other.currency}`,
			);
		}
	}

	// ===================================
	// STRING REPRESENTATION
	// ===================================

	/**
	 * Representação em string
	 */
	toString(): string {
		return this.format();
	}

	/**
	 * Para debug
	 */
	toDebugString(): string {
		return `Money(${this.amount} cents, ${this.currency})`;
	}
}
\n\n===== FILE: ./src/modules/billing/domain/enums/subscription-status.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { subscriptionStatus } from "../../../../infrastructure/database/drizzle/schema/enums";

export const subscriptionStatusSchema = createSelectSchema(subscriptionStatus);

export type SubscriptionStatus = z.infer<typeof subscriptionStatusSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/subscription-plan.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { subscriptionPlan } from "../../../../infrastructure/database/drizzle/schema/enums";

export const subscriptionPlanSchema = createSelectSchema(subscriptionPlan);

export type SubscriptionPlan = z.infer<typeof subscriptionPlanSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/payment-method-type.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { paymentMethodType } from "../../../../infrastructure/database/drizzle/schema/enums";

export const paymentMethodTypeSchema = createSelectSchema(paymentMethodType);

export type PaymentMethodType = z.infer<typeof paymentMethodTypeSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/transaction-type.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { transactionType } from "../../../../infrastructure/database/drizzle/schema/enums";

export const transactionTypeSchema = createSelectSchema(transactionType);

export type TransactionType = z.infer<typeof transactionTypeSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/transaction-status.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { transactionStatus } from "../../../../infrastructure/database/drizzle/schema/enums";

export const transactionStatusSchema = createSelectSchema(transactionStatus);

export type TransactionStatus = z.infer<typeof transactionStatusSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/billing-cycle.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { billingCycle } from "../../../../infrastructure/database/drizzle/schema/enums";

export const billingCycleSchema = createSelectSchema(billingCycle);

export type BillingCycle = z.infer<typeof billingCycleSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/invoice-status.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { invoiceStatus } from "../../../../infrastructure/database/drizzle/schema/enums";

export const invoiceStatusSchema = createSelectSchema(invoiceStatus);

export type InvoiceStatus = z.infer<typeof invoiceStatusSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/card-brand.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { cardBrand } from "../../../../infrastructure/database/drizzle/schema/enums";

export const cardBrandSchema = createSelectSchema(cardBrand);

export type CardBrand = z.infer<typeof cardBrandSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/payment-method-status.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { paymentMethodStatus } from "../../../../infrastructure/database/drizzle/schema/enums";

export const paymentMethodStatusSchema = createSelectSchema(paymentMethodStatus);

export type PaymentMethodStatus = z.infer<typeof paymentMethodStatusSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/collection-method.ts =====
import { createSelectSchema } from "drizzle-zod";
import type z from "zod";
import { collectionMethod } from "../../../../infrastructure/database/drizzle/schema/enums";

export const collectionMethodSchema = createSelectSchema(collectionMethod);

export type CollectionMethod = z.infer<typeof collectionMethodSchema>;
\n\n===== FILE: ./src/modules/billing/domain/enums/index.ts =====
export * from "./subscription-status";
export * from "./subscription-plan";
export * from "./payment-method-type";
export * from "./transaction-type";
export * from "./transaction-status";
export * from "./billing-cycle";
export * from "./invoice-status";
export * from "./card-brand";
export * from "./payment-method-status";
export * from "./collection-method";
\n\n===== FILE: ./src/modules/billing/domain/repositories/plan.repository.ts =====
import type { SubscriptionPlan } from "../enums";
import type { Plan } from "../entities/plan.entity";

export interface IPlanRepository {
	findById(id: string): Promise<Plan | null>;
	findByType(type: SubscriptionPlan): Promise<Plan | null>;
	findByProviderProductId(providerProductId: string): Promise<Plan | null>;
	findAllActive(): Promise<Plan[]>;
	findAllPublic(): Promise<Plan[]>;
	findDefault(): Promise<Plan | null>;

	save(plan: Plan): Promise<void>;
	delete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/billing/domain/repositories/subscription.repository.ts =====
import type { Subscription } from "../entities/subscription.entity";

export interface ISubscriptionRepository {
	findById(id: string): Promise<Subscription | null>;
	findByUserId(userId: string): Promise<Subscription | null>;
	findActiveByUserId(userId: string): Promise<Subscription | null>;
	findByProviderSubscriptionId(providerSubscriptionId: string): Promise<Subscription | null>;

	save(subscription: Subscription): Promise<void>;
	delete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/billing/domain/repositories/payment-method.repository.ts =====
import type { PaymentMethod } from "../entities/payment-method.entity";

export interface IPaymentMethodRepository {
	findById(id: string): Promise<PaymentMethod | null>;
	findByUserId(userId: string): Promise<PaymentMethod[]>;
	findDefaultByUserId(userId: string): Promise<PaymentMethod | null>;
	findByProviderPaymentMethodId(providerPaymentMethodId: string): Promise<PaymentMethod | null>;

	save(paymentMethod: PaymentMethod): Promise<void>;
	delete(id: string): Promise<void>;
}
\n\n===== FILE: ./src/modules/billing/domain/repositories/transaction.repository.ts =====
import type { Transaction } from "../entities/transaction.entity";

export interface ITransactionRepository {
	findById(id: string): Promise<Transaction | null>;
	findByUserId(userId: string): Promise<Transaction[]>;
	findBySubscriptionId(subscriptionId: string): Promise<Transaction[]>;
	findByProviderTransactionId(providerTransactionId: string): Promise<Transaction | null>;
	findByProviderPaymentIntentId(providerPaymentIntentId: string): Promise<Transaction | null>;

	save(transaction: Transaction): Promise<void>;
}
\n\n===== FILE: ./src/modules/billing/domain/repositories/invoice.repository.ts =====
import type { Invoice } from "../entities/invoice.entity";

export interface IInvoiceRepository {
	findById(id: string): Promise<Invoice | null>;
	findByUserId(userId: string): Promise<Invoice[]>;
	findBySubscriptionId(subscriptionId: string): Promise<Invoice[]>;
	findByNumber(number: string): Promise<Invoice | null>;
	findByProviderInvoiceId(providerInvoiceId: string): Promise<Invoice | null>;

	save(invoice: Invoice): Promise<void>;
}
\n\n===== FILE: ./src/modules/billing/domain/repositories/index.ts =====
export * from "./plan.repository";
export * from "./subscription.repository";
export * from "./payment-method.repository";
export * from "./transaction.repository";
export * from "./invoice.repository";
\n\n===== FILE: ./src/modules/billing/infrastructure/providers/payment-provider.interface.ts =====
import type { BillingCycle, PaymentMethodType } from "../../domain/enums";

// ===================================
// CUSTOMER
// ===================================

export interface CreateCustomerInput {
	email: string;
	name: string;
	taxId?: string;
	metadata?: Record<string, string>;
}

export interface ProviderCustomer {
	providerCustomerId: string;
	email: string;
	name: string;
}

// ===================================
// PRODUCT & PRICE
// ===================================

export interface CreateProductInput {
	name: string;
	description?: string;
	metadata?: Record<string, string>;
}

export interface CreatePriceInput {
	providerProductId: string;
	amountInCents: number;
	currency: string;
	billingCycle: BillingCycle;
}

export interface ProviderProduct {
	providerProductId: string;
	name: string;
}

export interface ProviderPrice {
	providerPriceId: string;
	providerProductId: string;
	amountInCents: number;
	currency: string;
}

// ===================================
// SUBSCRIPTION
// ===================================

export interface CreateSubscriptionInput {
	providerCustomerId: string;
	providerPriceId: string;
	paymentMethodId?: string;
	trialDays?: number;
	metadata?: Record<string, string>;
}

export interface UpdateSubscriptionInput {
	providerSubscriptionId: string;
	providerPriceId?: string;
	cancelAtPeriodEnd?: boolean;
	metadata?: Record<string, string>;
}

export interface ProviderSubscription {
	providerSubscriptionId: string;
	providerCustomerId: string;
	providerPriceId: string;
	status: string;
	currentPeriodStart: Date;
	currentPeriodEnd: Date;
	cancelAtPeriodEnd: boolean;
	trialStart: Date | null;
	trialEnd: Date | null;
	latestInvoiceId: string | null;
	defaultPaymentMethodId: string | null;
}

// ===================================
// PAYMENT METHOD
// ===================================

export interface AttachPaymentMethodInput {
	providerPaymentMethodId: string;
	providerCustomerId: string;
}

export interface ProviderPaymentMethod {
	providerPaymentMethodId: string;
	type: PaymentMethodType;
	card?: {
		brand: string;
		last4: string;
		expiryMonth: number;
		expiryYear: number;
		fingerprint: string | null;
		funding: string;
		country: string | null;
	};
}

// ===================================
// CHECKOUT / PAYMENT INTENT
// ===================================

export interface CreatePaymentIntentInput {
	amountInCents: number;
	currency: string;
	providerCustomerId: string;
	paymentMethodTypes: PaymentMethodType[];
	providerPaymentMethodId?: string;
	description?: string;
	metadata?: Record<string, string>;
}

export interface ProviderPaymentIntent {
	providerPaymentIntentId: string;
	clientSecret: string;
	status: string;
	amountInCents: number;
	currency: string;
}

// ===================================
// REFUND
// ===================================

export interface CreateRefundInput {
	providerPaymentIntentId?: string;
	providerChargeId?: string;
	amountInCents?: number;
	reason?: string;
	metadata?: Record<string, string>;
}

export interface ProviderRefund {
	providerRefundId: string;
	amountInCents: number;
	status: string;
	currency: string;
}

// ===================================
// INVOICE
// ===================================

export interface CreateInvoiceInput {
	providerCustomerId: string;
	description?: string;
	metadata?: Record<string, string>;
	autoAdvance?: boolean;
}

export interface AddInvoiceItemInput {
	providerCustomerId: string;
	providerInvoiceId: string;
	amountInCents: number;
	currency: string;
	description: string;
}

export interface ProviderInvoice {
	providerInvoiceId: string;
	status: string;
	amountDue: number;
	amountPaid: number;
	amountRemaining: number;
	currency: string;
	hostedInvoiceUrl: string | null;
	invoicePdf: string | null;
}

// ===================================
// WEBHOOK
// ===================================

export interface WebhookEvent {
	id: string;
	type: string;
	data: Record<string, unknown>;
	createdAt: Date;
}

// ===================================
// PAYMENT PROVIDER INTERFACE (PORT)
// ===================================

export interface IPaymentProvider {
	readonly providerName: string;

	// Customer
	createCustomer(input: CreateCustomerInput): Promise<ProviderCustomer>;
	getCustomer(providerCustomerId: string): Promise<ProviderCustomer | null>;
	updateCustomer(
		providerCustomerId: string,
		input: Partial<CreateCustomerInput>,
	): Promise<ProviderCustomer>;
	deleteCustomer(providerCustomerId: string): Promise<void>;

	// Product & Price
	createProduct(input: CreateProductInput): Promise<ProviderProduct>;
	createPrice(input: CreatePriceInput): Promise<ProviderPrice>;
	deactivateProduct(providerProductId: string): Promise<void>;

	// Subscription
	createSubscription(
		input: CreateSubscriptionInput,
	): Promise<ProviderSubscription>;
	getSubscription(
		providerSubscriptionId: string,
	): Promise<ProviderSubscription | null>;
	updateSubscription(
		input: UpdateSubscriptionInput,
	): Promise<ProviderSubscription>;
	cancelSubscription(
		providerSubscriptionId: string,
		immediately?: boolean,
	): Promise<ProviderSubscription>;
	pauseSubscription(
		providerSubscriptionId: string,
	): Promise<ProviderSubscription>;
	resumeSubscription(
		providerSubscriptionId: string,
	): Promise<ProviderSubscription>;

	// Payment Method
	attachPaymentMethod(
		input: AttachPaymentMethodInput,
	): Promise<ProviderPaymentMethod>;
	detachPaymentMethod(providerPaymentMethodId: string): Promise<void>;
	listPaymentMethods(
		providerCustomerId: string,
	): Promise<ProviderPaymentMethod[]>;

	// Payment Intent (for one-time payments: PIX, boleto, etc.)
	createPaymentIntent(
		input: CreatePaymentIntentInput,
	): Promise<ProviderPaymentIntent>;
	getPaymentIntent(
		providerPaymentIntentId: string,
	): Promise<ProviderPaymentIntent | null>;
	cancelPaymentIntent(providerPaymentIntentId: string): Promise<void>;

	// Refund
	createRefund(input: CreateRefundInput): Promise<ProviderRefund>;

	// Invoice
	createInvoice(input: CreateInvoiceInput): Promise<ProviderInvoice>;
	addInvoiceItem(input: AddInvoiceItemInput): Promise<void>;
	finalizeInvoice(providerInvoiceId: string): Promise<ProviderInvoice>;
	voidInvoice(providerInvoiceId: string): Promise<ProviderInvoice>;

	// Webhook
	constructWebhookEvent(
		payload: string | Buffer,
		signature: string,
	): WebhookEvent;
}
\n\n===== FILE: ./src/modules/billing/infrastructure/webhooks/payment-webhook.handler.ts =====
\n\n===== FILE: ./src/modules/billing/application/use-cases/create-subscription.use-case.ts =====
\n\n===== FILE: ./src/core/errors/index.ts =====
export class DomainError extends Error {
	constructor(
		message: string,
		public readonly code: string,
		public readonly statusCode: number = 400,
		public readonly details?: Record<string, any>,
	) {
		super(message);
		this.name = "DomainError";
	}
	// Método para serializar pra logs
	toJSON() {
		return {
			name: this.name,
			message: this.message,
			code: this.code,
			statusCode: this.statusCode,
			details: this.details,
			stack: this.stack,
		};
	}
}

export class ValidationError extends DomainError {
	constructor(message: string, details?: Record<string, any>) {
		super(message, "VALIDATION_ERROR", 400, details);
	}
}

export class NotFoundError extends DomainError {
	constructor(resource: string) {
		super(`${resource} não encontrado`, "NOT_FOUND", 404);
	}
}

export class UnauthorizedError extends DomainError {
	constructor(resource?: string) {
		super(resource ?? "Não autorizado", "UNAUTHORIZED", 401);
	}
}

export class BusinessError extends DomainError {
	constructor(message: string, code = "BUSINESS_ERROR") {
		super(message, code, 400);
	}
}

export class InternalError extends DomainError {
	constructor(message: string = "Erro interno inesperado") {
		super(message, "INTERNAL_ERROR", 500);
	}
}

export class ConflictError extends DomainError {
	constructor(message = "Conflito de dados detectado") {
		super(message, "CONFLICT_ERROR", 409); // 409 é o HTTP status para conflitos
	}
}

export class TooManyRequestsError extends Error {
	statusCode = 429;

	constructor(message = "Muitas requisições. Tente novamente mais tarde.") {
		super(message);
		this.name = "TooManyRequestsError";
	}
}
\n\n===== FILE: ./src/core/contracts/base-repository.ts =====
import type { FastifyBaseLogger } from "fastify";
import type { DrizzleClient } from "../../infrastructure/database/drizzle/client";
import type { IUnitOfWork } from "../../infrastructure/database/drizzle/unit-of-work/unit-of-work.interface";

export interface BaseRepositoryDependencies {
	unitOfWork: IUnitOfWork;
	logger: FastifyBaseLogger;
}

export abstract class BaseRepository {
	protected readonly unitOfWork: IUnitOfWork;
	protected readonly logger: FastifyBaseLogger;

	constructor({ unitOfWork, logger }: BaseRepositoryDependencies) {
		this.unitOfWork = unitOfWork;
		this.logger = logger;
	}

	// Getter mágico: Sempre pega o cliente certo (Transação ou Conexão Pura
	protected get db(): DrizzleClient {
		return this.unitOfWork.getTransactionClient() as DrizzleClient;
	}
}
\n\n===== FILE: ./src/core/contracts/base-use-case.ts =====
import type { FastifyBaseLogger } from "fastify";
import type { IUnitOfWork } from "../../infrastructure/database/drizzle/unit-of-work/unit-of-work.interface.js";

// 1. Interface de dependências padrão para todos os UseCases
export interface UseCaseDependencies {
	unitOfWork: IUnitOfWork;
	logger: FastifyBaseLogger; // Confia em mim, você vai querer log em todo UseCase
}

export interface IUseCase<Input, Output, Metadata = never> {
	execute(input: Input, metadata?: Metadata): Promise<Output>;
}

/**
 * ✅ UseCase base com UnitOfWork e Logger integrados
 */
export abstract class TransactionalUseCase<Input, Output, Metadata = undefined>
	implements IUseCase<Input, Output, Metadata>
{
	protected readonly unitOfWork: IUnitOfWork;
	protected readonly logger: FastifyBaseLogger;

	// Recebemos o objeto de dependências (Padrão Awilix Proxy)
	constructor({ unitOfWork, logger }: UseCaseDependencies) {
		this.unitOfWork = unitOfWork;
		this.logger = logger;
	}

	// Simplificamos a assinatura para facilitar a implementação nas classes filhas
	abstract execute(input: Input, metadata?: Metadata): Promise<Output>;

	/**
	 * Helper para transações
	 */
	protected async runInTransaction<T>(callback: () => Promise<T>): Promise<T> {
		return this.unitOfWork.run(callback);
	}
}
\n\n===== FILE: ./src/core/contracts/base-task.ts =====
import type { FastifyBaseLogger } from "fastify";
import type { IUnitOfWork } from "../../infrastructure/database/drizzle/unit-of-work/unit-of-work.interface.js";

export interface TaskDependencies {
	unitOfWork: IUnitOfWork;
	logger: FastifyBaseLogger;
}

export abstract class BaseTask {
	protected readonly unitOfWork: IUnitOfWork;
	protected readonly logger: FastifyBaseLogger;

	constructor({ unitOfWork, logger }: TaskDependencies) {
		this.unitOfWork = unitOfWork;
		this.logger = logger;
	}

	// O método que o cron vai chamar
	abstract handleCron(): Promise<void>;

	/**
	 * Helper para transações dentro da Task, caso precise
	 */
	protected async runInTransaction<T>(callback: () => Promise<T>): Promise<T> {
		return this.unitOfWork.run(callback);
	}
}
\n\n===== FILE: ./drizzle.config.ts =====
import { defineConfig } from "drizzle-kit";
import { env } from "./src/shared/config/index.js";

export default defineConfig({
	schema: "./src/infrastructure/database/drizzle/schema",
	out: "./src/infrastructure/database/drizzle/migrations",
	dialect: "postgresql",
	dbCredentials: {
		url: env.DATABASE_URL,
	},
	casing: "snake_case",
});
\n\n===== FILE: ./README.md =====
# api

To install dependencies:

```bash
bun install
```

To run:

```bash
bun run index.ts
```

This project was created using `bun init` in bun v1.3.5. [Bun](https://bun.com) is a fast all-in-one JavaScript runtime.
\n\n===== FILE: ./railway.json =====
{
    "$schema": "https://railway.com/railway.schema.json",
    "build": {
        "builder": "RAILPACK",
        "buildCommand": "bun x turbo run build --filter=api",
        "watchPatterns": [
            "/apps/api/**",
            "/packages/**"
        ]
    },
    "deploy": {
        "startCommand": "bun x turbo run start --filter=api",
        "healthcheckPath": "/health",
        "restartPolicyType": "ALWAYS"
    }
}