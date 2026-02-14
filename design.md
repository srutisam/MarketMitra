# Design Document: MarketMitra Decision Support System

## Overview

MarketMitra is a cloud-native, microservices-based AI decision support system designed to empower rural micro-entrepreneurs in India. The system architecture prioritizes scalability, low-bandwidth optimization, multilingual support, and explainable AI.

### Core Design Principles

1. **Voice-First Design**: Primary interaction through voice in local languages, with text as fallback
2. **Offline-Capable**: Core functionality available with cached data during connectivity issues
3. **Explainability by Default**: Every AI recommendation includes clear, jargon-free explanations
4. **Progressive Disclosure**: Simple interface with progressively deeper details on demand
5. **Bandwidth Optimization**: Aggressive caching, compression, and adaptive quality
6. **Privacy by Design**: End-to-end encryption, data minimization, user control

### Technology Stack

**Frontend:**
- Progressive Web App (PWA) using React Native Web
- Offline-first architecture with service workers
- Voice interaction using Web Speech API with fallback to cloud STT/TTS
- Lightweight UI framework optimized for low-end devices

**Backend:**
- Microservices architecture using Node.js/Python
- API Gateway (Kong or AWS API Gateway)
- Message queue (RabbitMQ or AWS SQS) for async processing
- Container orchestration (Kubernetes)

**AI/ML:**
- Time-series forecasting: Prophet, ARIMA, LSTM (TensorFlow/PyTorch)
- Optimization: SciPy, OR-Tools for inventory and pricing
- Explainability: SHAP, LIME for model interpretability
- NLP: Multilingual models (IndicBERT, mBERT) for text understanding
- Speech: Google Cloud Speech-to-Text/Text-to-Speech with Indic language support

**Data Storage:**
- PostgreSQL for transactional data
- TimescaleDB for time-series data
- Redis for caching and session management
- S3-compatible object storage for model artifacts and media

**Infrastructure:**
- Cloud provider: AWS/GCP (multi-region deployment)
- CDN: CloudFlare for static assets and API caching
- Monitoring: Prometheus, Grafana, ELK stack
- CI/CD: GitHub Actions, ArgoCD

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   PWA App    │  │  Voice Bot   │  │   USSD/SMS   │          │
│  │  (Primary)   │  │  (WhatsApp)  │  │  (Fallback)  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Authentication │ Rate Limiting │ Request Routing │ Cache  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Application Services Layer                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   User       │  │   Business   │  │   Voice      │          │
│  │   Service    │  │   Profile    │  │   Service    │          │
│  │              │  │   Service    │  │              │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Forecast   │  │   Pricing    │  │  Inventory   │          │
│  │   Service    │  │   Service    │  │   Service    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │     PnL      │  │ Explanation  │  │  Analytics   │          │
│  │  Simulator   │  │   Service    │  │   Service    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      AI/ML Services Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Model      │  │   Feature    │  │  Explanation │          │
│  │   Serving    │  │  Engineering │  │   Generator  │          │
│  │  (TF Serve)  │  │   Pipeline   │  │   (SHAP)     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Model      │  │   Training   │  │  Monitoring  │          │
│  │  Registry    │  │   Pipeline   │  │  & Drift     │          │
│  │  (MLflow)    │  │  (Airflow)   │  │  Detection   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Data Layer                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  PostgreSQL  │  │ TimescaleDB  │  │    Redis     │          │
│  │ (Transact.)  │  │(Time-series) │  │   (Cache)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   S3/Blob    │  │  Message     │  │   Search     │          │
│  │   Storage    │  │   Queue      │  │(Elasticsearch)│          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   External Data Sources                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   e-NAM API  │  │  Agmarknet   │  │   Weather    │          │
│  │              │  │     API      │  │     API      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### Service Communication Patterns

**Synchronous (REST/gRPC):**
- Client to API Gateway
- API Gateway to Application Services
- Application Services to AI/ML Services (for real-time predictions)

**Asynchronous (Message Queue):**
- Model training triggers
- Batch forecast generation
- Analytics aggregation
- Notification delivery

**Event-Driven:**
- Business profile updates trigger forecast recalculation
- Market data updates trigger price recommendation refresh
- Model performance degradation triggers retraining

## Components and Interfaces

### 1. Voice Service

**Responsibilities:**
- Speech-to-text conversion for Indic languages
- Text-to-speech generation
- Voice command parsing and intent recognition
- Conversation state management

**Key Interfaces:**

```typescript
interface VoiceService {
  // Convert speech audio to text
  transcribe(
    audioData: Buffer,
    language: IndicLanguage,
    userId: string
  ): Promise<TranscriptionResult>;

  // Convert text to speech audio
  synthesize(
    text: string,
    language: IndicLanguage,
    voiceProfile: VoiceProfile
  ): Promise<AudioBuffer>;

  // Parse user intent from transcribed text
  parseIntent(
    text: string,
    language: IndicLanguage,
    context: ConversationContext
  ): Promise<UserIntent>;

  // Manage conversation flow
  handleConversation(
    intent: UserIntent,
    context: ConversationContext
  ): Promise<ConversationResponse>;
}

interface TranscriptionResult {
  text: string;
  confidence: number;
  language: IndicLanguage;
  alternates?: string[];
}

interface UserIntent {
  action: IntentAction; // FORECAST, PRICE, INVENTORY, SIMULATE, etc.
  entities: Map<string, any>; // product, timeframe, etc.
  confidence: number;
}

type IndicLanguage = 'hi' | 'ta' | 'te' | 'bn' | 'mr' | 'gu';
type IntentAction = 'FORECAST_DEMAND' | 'OPTIMIZE_PRICE' | 'MANAGE_INVENTORY' | 
                    'SIMULATE_PNL' | 'VIEW_REPORT' | 'UPDATE_PROFILE';
```

### 2. Forecast Service

**Responsibilities:**
- Generate demand forecasts using time-series models
- Manage forecast horizons (7, 14, 30 days)
- Calculate confidence intervals
- Handle insufficient data scenarios

**Key Interfaces:**

```typescript
interface ForecastService {
  // Generate demand forecast for a product
  generateForecast(
    request: ForecastRequest
  ): Promise<ForecastResult>;

  // Get historical forecast accuracy
  getForecastAccuracy(
    productId: string,
    userId: string,
    timeRange: DateRange
  ): Promise<AccuracyMetrics>;

  // Trigger model retraining
  triggerRetraining(
    userId: string,
    productId?: string
  ): Promise<RetrainingJob>;
}

interface ForecastRequest {
  userId: string;
  productId: string;
  horizonDays: 7 | 14 | 30;
  includeConfidenceIntervals: boolean;
  includeExplanation: boolean;
}

interface ForecastResult {
  productId: string;
  forecasts: DailyForecast[];
  confidence: number;
  modelUsed: string;
  generatedAt: Date;
  explanation?: Explanation;
  warnings?: string[];
}

interface DailyForecast {
  date: Date;
  predictedDemand: number;
  lowerBound: number;
  upperBound: number;
  confidence: number;
}

interface AccuracyMetrics {
  mape: number; // Mean Absolute Percentage Error
  rmse: number; // Root Mean Squared Error
  mae: number;  // Mean Absolute Error
  sampleSize: number;
}
```

### 3. Pricing Service

**Responsibilities:**
- Calculate optimal pricing recommendations
- Analyze price elasticity
- Monitor competitor prices
- Generate dynamic pricing strategies

**Key Interfaces:**

```typescript
interface PricingService {
  // Get pricing recommendation
  recommendPrice(
    request: PricingRequest
  ): Promise<PricingRecommendation>;

  // Analyze price elasticity
  analyzePriceElasticity(
    productId: string,
    userId: string
  ): Promise<ElasticityAnalysis>;

  // Monitor competitor prices
  getCompetitorPrices(
    productId: string,
    location: Location
  ): Promise<CompetitorPrice[]>;
}

interface PricingRequest {
  userId: string;
  productId: string;
  currentPrice?: number;
  costPrice: number;
  targetMargin?: number;
  includeExplanation: boolean;
}

interface PricingRecommendation {
  productId: string;
  recommendations: PricePoint[];
  currentMarketPrice: number;
  competitorPriceRange: { min: number; max: number };
  explanation?: Explanation;
  dynamicPricingTriggers?: PricingTrigger[];
}

interface PricePoint {
  price: number;
  type: 'minimum' | 'optimal' | 'maximum';
  expectedVolume: number;
  expectedRevenue: number;
  expectedProfit: number;
  confidence: number;
}

interface ElasticityAnalysis {
  elasticityCoefficient: number;
  interpretation: 'elastic' | 'inelastic' | 'unit_elastic';
  priceChangeImpact: PriceImpact[];
}

interface PricingTrigger {
  condition: string;
  recommendedAction: string;
  threshold: number;
}
```

### 4. Inventory Service

**Responsibilities:**
- Calculate optimal inventory levels
- Determine reorder points and quantities
- Account for perishability and storage constraints
- Generate stockout warnings

**Key Interfaces:**

```typescript
interface InventoryService {
  // Get inventory recommendations
  recommendInventory(
    request: InventoryRequest
  ): Promise<InventoryRecommendation>;

  // Calculate reorder point
  calculateReorderPoint(
    productId: string,
    userId: string,
    leadTimeDays: number
  ): Promise<ReorderPoint>;

  // Check stockout risk
  assessStockoutRisk(
    productId: string,
    userId: string,
    currentStock: number
  ): Promise<StockoutRisk>;
}

interface InventoryRequest {
  userId: string;
  productId: string;
  currentStock: number;
  leadTimeDays: number;
  storageCapacity?: number;
  capitalConstraint?: number;
  includeExplanation: boolean;
}

interface InventoryRecommendation {
  productId: string;
  optimalStockLevel: number;
  reorderPoint: number;
  reorderQuantity: number;
  safetyStock: number;
  expectedHoldingCost: number;
  expectedStockoutCost: number;
  explanation?: Explanation;
  warnings?: string[];
}

interface ReorderPoint {
  quantity: number;
  triggerDate: Date;
  confidence: number;
}

interface StockoutRisk {
  probability: number;
  expectedDate: Date;
  severity: 'low' | 'medium' | 'high';
  recommendedAction: string;
}
```

### 5. PnL Simulator Service

**Responsibilities:**
- Simulate profit and loss scenarios
- Model best/expected/worst cases
- Calculate cash flow timing
- Identify risk factors

**Key Interfaces:**

```typescript
interface PnLSimulatorService {
  // Simulate a business scenario
  simulateScenario(
    request: SimulationRequest
  ): Promise<SimulationResult>;

  // Compare multiple scenarios
  compareScenarios(
    scenarios: SimulationRequest[]
  ): Promise<ScenarioComparison>;
}

interface SimulationRequest {
  userId: string;
  productId: string;
  price: number;
  volume: number;
  costPrice: number;
  fixedCosts: number;
  variableCosts: number;
  timeHorizonDays: number;
  includeExplanation: boolean;
}

interface SimulationResult {
  scenario: SimulationRequest;
  outcomes: {
    bestCase: FinancialOutcome;
    expectedCase: FinancialOutcome;
    worstCase: FinancialOutcome;
  };
  cashFlowTimeline: CashFlowPoint[];
  riskFactors: RiskFactor[];
  explanation?: Explanation;
}

interface FinancialOutcome {
  revenue: number;
  totalCosts: number;
  profit: number;
  profitMargin: number;
  probability: number;
}

interface CashFlowPoint {
  date: Date;
  inflow: number;
  outflow: number;
  netCashFlow: number;
  cumulativeCashFlow: number;
}

interface RiskFactor {
  factor: string;
  severity: 'low' | 'medium' | 'high';
  impact: number;
  mitigation: string;
}

interface ScenarioComparison {
  scenarios: SimulationResult[];
  recommendation: number; // index of recommended scenario
  comparisonMetrics: ComparisonMetric[];
}
```

### 6. Explanation Service

**Responsibilities:**
- Generate human-readable explanations for AI predictions
- Use SHAP/LIME for feature importance
- Adapt explanation complexity to user literacy level
- Support multilingual explanations

**Key Interfaces:**

```typescript
interface ExplanationService {
  // Generate explanation for a prediction
  explainPrediction(
    request: ExplanationRequest
  ): Promise<Explanation>;

  // Get feature importance
  getFeatureImportance(
    modelId: string,
    predictionId: string
  ): Promise<FeatureImportance[]>;
}

interface ExplanationRequest {
  predictionType: 'forecast' | 'pricing' | 'inventory' | 'simulation';
  predictionId: string;
  userId: string;
  language: IndicLanguage;
  detailLevel: 'simple' | 'detailed' | 'technical';
}

interface Explanation {
  summary: string; // One-sentence explanation
  keyFactors: ExplanationFactor[];
  detailedExplanation?: string;
  visualAids?: VisualizationData[];
  confidence: number;
  uncertaintyNote?: string;
}

interface ExplanationFactor {
  factor: string;
  impact: 'positive' | 'negative' | 'neutral';
  magnitude: number; // 0-100
  description: string;
}

interface FeatureImportance {
  featureName: string;
  importance: number;
  direction: 'increases' | 'decreases';
  humanReadableName: string;
}
```

### 7. Business Profile Service

**Responsibilities:**
- Manage entrepreneur business data
- Validate profile completeness
- Handle profile updates and versioning
- Trigger dependent service updates

**Key Interfaces:**

```typescript
interface BusinessProfileService {
  // Create new business profile
  createProfile(
    profile: BusinessProfile
  ): Promise<ProfileCreationResult>;

  // Update existing profile
  updateProfile(
    userId: string,
    updates: Partial<BusinessProfile>
  ): Promise<ProfileUpdateResult>;

  // Get profile
  getProfile(
    userId: string
  ): Promise<BusinessProfile>;

  // Validate profile completeness
  validateProfile(
    profile: BusinessProfile
  ): Promise<ValidationResult>;
}

interface BusinessProfile {
  userId: string;
  businessName: string;
  businessType: BusinessType;
  location: Location;
  products: Product[];
  suppliers: Supplier[];
  storageCapacity?: number;
  capitalAvailable?: number;
  preferredLanguage: IndicLanguage;
  createdAt: Date;
  updatedAt: Date;
}

interface Product {
  id: string;
  name: string;
  category: string;
  unit: string;
  costPrice: number;
  sellingPrice: number;
  perishable: boolean;
  shelfLifeDays?: number;
  historicalSales?: SalesDataPoint[];
}

interface SalesDataPoint {
  date: Date;
  quantity: number;
  price: number;
  revenue: number;
}

interface ValidationResult {
  isComplete: boolean;
  missingFields: string[];
  warnings: string[];
  completenessScore: number; // 0-100
}

type BusinessType = 'grocery' | 'dairy' | 'handicraft' | 'agriculture' | 
                    'retail' | 'services' | 'other';
```

## Data Models

### Core Entities

**User:**
```typescript
interface User {
  id: string;
  phoneNumber: string;
  name: string;
  preferredLanguage: IndicLanguage;
  registrationDate: Date;
  lastActiveDate: Date;
  onboardingCompleted: boolean;
  subscriptionTier: 'free' | 'basic' | 'premium';
}
```

**Forecast:**
```typescript
interface Forecast {
  id: string;
  userId: string;
  productId: string;
  modelVersion: string;
  horizonDays: number;
  predictions: DailyForecast[];
  confidence: number;
  generatedAt: Date;
  expiresAt: Date;
  actualOutcomes?: ActualOutcome[];
}

interface ActualOutcome {
  date: Date;
  actualDemand: number;
  predictedDemand: number;
  error: number;
  percentageError: number;
}
```

**Recommendation:**
```typescript
interface Recommendation {
  id: string;
  userId: string;
  type: 'pricing' | 'inventory' | 'forecast';
  productId: string;
  recommendation: any; // Type-specific recommendation data
  explanation: Explanation;
  confidence: number;
  createdAt: Date;
  expiresAt: Date;
  userAction?: UserAction;
}

interface UserAction {
  action: 'accepted' | 'rejected' | 'modified';
  timestamp: Date;
  outcome?: ActionOutcome;
}

interface ActionOutcome {
  measuredAt: Date;
  metrics: Map<string, number>;
  success: boolean;
}
```

**Model Metadata:**
```typescript
interface ModelMetadata {
  id: string;
  name: string;
  version: string;
  type: 'forecast' | 'pricing' | 'inventory';
  algorithm: string;
  hyperparameters: Map<string, any>;
  trainingDate: Date;
  performanceMetrics: Map<string, number>;
  status: 'active' | 'deprecated' | 'testing';
  applicableRegions?: string[];
  applicableCategories?: string[];
}
```

### Database Schema Design

**Time-Series Data (TimescaleDB):**
- Sales transactions (partitioned by date)
- Market prices (partitioned by date)
- Forecast predictions (partitioned by date)
- Model performance metrics (partitioned by date)

**Transactional Data (PostgreSQL):**
- Users and authentication
- Business profiles
- Products and suppliers
- Recommendations and user actions

**Cache Data (Redis):**
- Session state
- Frequently accessed forecasts
- Market data snapshots
- API rate limiting counters

### Data Retention Policy

- **Raw sales data**: 2 years
- **Aggregated sales data**: Indefinite
- **Forecasts**: 90 days after expiration
- **User sessions**: 30 days
- **Logs**: 90 days (hot), 1 year (cold)
- **Model artifacts**: All versions (compressed)


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I identified several areas where properties can be consolidated:

**Structural Requirements Consolidation:**
- Multiple requirements specify that responses must include explanations (1.5, 2.4, 3.5, 6.1). These can be combined into one comprehensive property about explanation presence.
- Multiple requirements specify that responses must include specific fields (confidence scores, price points, reorder quantities). These can be grouped by service type.

**Conditional Logic Consolidation:**
- Several requirements follow the pattern "when X exceeds threshold, trigger Y" (2.5, 3.4, 4.5, 6.5, 10.2, 10.4). These share similar testing patterns but validate different thresholds.

**Performance Requirements:**
- Multiple performance requirements (5.2, 8.5, 14.1, 14.3) test timing under different conditions. These should remain separate as they test different scenarios.

After reflection, I've consolidated redundant properties and ensured each remaining property provides unique validation value.

### Forecast Service Properties

**Property 1: Multi-horizon forecast generation**
*For any* valid product and business profile, when requesting a demand forecast, the system should return predictions for all three time horizons (7, 14, and 30 days) with each horizon containing daily forecasts.
**Validates: Requirements 1.1**

**Property 2: Forecast confidence scoring**
*For any* generated forecast, the response must include a confidence score between 0 and 100, and this score should correlate with actual forecast accuracy when measured against outcomes.
**Validates: Requirements 1.3**

**Property 3: Forecast explanation completeness**
*For any* forecast response, an explanation object must be present containing at least one key factor with impact magnitude and description.
**Validates: Requirements 1.5**

**Property 4: Low confidence uncertainty warnings**
*For any* prediction with confidence score below 60%, the response must include an explicit uncertainty warning and caution recommendation.
**Validates: Requirements 6.5**

### Pricing Service Properties

**Property 5: Three-tier pricing structure**
*For any* pricing request, the recommendation must include exactly three price points (minimum, optimal, maximum), each with expected volume, revenue, and profit projections.
**Validates: Requirements 2.2**

**Property 6: Pricing explanation completeness**
*For any* pricing recommendation, the explanation must address all three factors: profit margin, sales volume, and competitive positioning.
**Validates: Requirements 2.4**

**Property 7: Market change alerting**
*For any* product where competitor prices or input costs change by more than 15%, the system should generate a proactive alert within 24 hours of the change.
**Validates: Requirements 2.5**

### Inventory Service Properties

**Property 8: Inventory recommendation completeness**
*For any* inventory request, the recommendation must include optimal stock level, reorder point, reorder quantity, and safety stock values, all as non-negative numbers.
**Validates: Requirements 3.1, 3.3**

**Property 9: Perishability impact on inventory**
*For any* two products with identical demand patterns but different perishability (one perishable, one not), the perishable product should receive a lower optimal stock level recommendation.
**Validates: Requirements 3.2**

**Property 10: Stockout risk warnings**
*For any* inventory assessment where stockout probability exceeds 20%, the response must include a warning flag and at least one recommended action.
**Validates: Requirements 3.4**

**Property 11: Inventory explanation completeness**
*For any* inventory recommendation, the explanation must address all three trade-offs: holding costs, stockout costs, and capital requirements.
**Validates: Requirements 3.5**

### PnL Simulation Properties

**Property 12: Three-scenario simulation**
*For any* valid business scenario input, the simulation must return exactly three outcomes (best-case, expected, worst-case), each with revenue, costs, profit, and probability values.
**Validates: Requirements 4.1, 4.2**

**Property 13: Scenario comparison recommendation**
*For any* set of two or more scenarios submitted for comparison, the system must identify and return the index of the recommended optimal scenario with justification.
**Validates: Requirements 4.3**

**Property 14: Cash flow timeline presence**
*For any* simulation result, a cash flow timeline must be present with at least one data point containing inflow, outflow, net cash flow, and cumulative cash flow values.
**Validates: Requirements 4.4**

**Property 15: High-risk scenario flagging**
*For any* simulation where the worst-case outcome has greater than 30% probability of loss, the response must include a risk flag and at least one mitigation strategy.
**Validates: Requirements 4.5**

### Voice Interface Properties

**Property 16: Multilingual transcription support**
*For any* audio input in a supported language (Hindi, Tamil, Telugu, Bengali, Marathi, Gujarati), the voice interface must return a transcription result with text and confidence score.
**Validates: Requirements 5.1**

**Property 17: Language consistency in responses**
*For any* voice query in language L, the system response must be in the same language L, verifiable by checking the response language field.
**Validates: Requirements 5.2**

**Property 18: Low-confidence clarification**
*For any* voice transcription with confidence below 80%, the system must request clarification before processing the intent.
**Validates: Requirements 5.3**

### Explainability Properties

**Property 19: Universal explanation presence**
*For any* recommendation of any type (forecast, pricing, inventory, simulation), the response must include an explanation object with a non-empty summary field.
**Validates: Requirements 6.1**

**Property 20: Ranked factor identification**
*For any* explanation, the key factors list must contain between 3 and 5 factors, each with a factor name, impact direction, magnitude, and description.
**Validates: Requirements 6.2**

**Property 21: Visual aid inclusion**
*For any* explanation, at least one visual aid (chart data, icon reference, or diagram) should be included to enhance understanding.
**Validates: Requirements 6.4**

### Business Profile Properties

**Property 22: Profile validation completeness**
*For any* business profile, validation must return a completeness score between 0 and 100, and profiles with scores below 70 must include a list of missing critical fields.
**Validates: Requirements 7.2**

**Property 23: Profile update triggers recalculation**
*For any* business profile update that changes product costs, prices, or demand data, all active recommendations for affected products must be marked for recalculation within 5 minutes.
**Validates: Requirements 7.4**

### Low-Bandwidth Mode Properties

**Property 24: Automatic bandwidth mode switching**
*For any* request where detected bandwidth falls below 2G speeds (50 kbps), the system must automatically enable low-bandwidth mode, verifiable by response metadata.
**Validates: Requirements 8.1**

**Property 25: Low-bandwidth mode behavior**
*For any* response in low-bandwidth mode, text content must be prioritized over audio, and response payload size must be at least 30% smaller than standard mode for equivalent data.
**Validates: Requirements 8.2**

**Property 26: Offline request queuing**
*For any* request made while offline, the request must be added to a queue and processed when connectivity is restored, with the response delivered to the user.
**Validates: Requirements 8.3**

**Property 27: Cached data accessibility**
*For any* previously fetched forecast or recommendation, when accessed offline, the cached version must be returned with a timestamp indicating when it was generated.
**Validates: Requirements 8.4**

**Property 28: 2G performance bounds**
*For any* core operation (forecast retrieval, price check) on a simulated 2G connection, the operation must complete within 10 seconds at p95 latency.
**Validates: Requirements 8.5**

### Data Privacy Properties

**Property 29: Training data anonymization**
*For any* dataset used for model training, individual entrepreneur identifiers (user ID, phone number, name) must not be present in the training data.
**Validates: Requirements 9.5**

### Model Performance Properties

**Property 30: Forecast accuracy tracking**
*For any* product with at least 30 days of historical data, the system must calculate and store monthly MAPE (Mean Absolute Percentage Error) for forecast accuracy.
**Validates: Requirements 10.1**

**Property 31: Accuracy-triggered retraining**
*For any* product category where forecast accuracy drops below 70% MAPE, a model retraining workflow must be triggered within 24 hours.
**Validates: Requirements 10.2**

**Property 32: Prediction outcome logging**
*For any* prediction made by the system, both the prediction and the actual outcome (when available) must be logged with timestamps for continuous learning.
**Validates: Requirements 10.3**

**Property 33: Model drift detection and response**
*For any* model where accuracy degrades by more than 10% compared to baseline, an administrator alert must be generated and confidence scores for that model must be reduced by at least 10 points.
**Validates: Requirements 10.4**

**Property 34: Segmented performance metrics**
*For any* model, performance metrics must be tracked separately for each supported language and geographic region, with at least one metric per segment.
**Validates: Requirements 10.5**

### Responsible AI Properties

**Property 35: Protected attribute exclusion**
*For any* prediction model's feature set, protected attributes (gender, caste, religion) must not appear as direct input features.
**Validates: Requirements 11.3**

**Property 36: Training data diversity**
*For any* training dataset, demographic representation must be within 20% of the target population distribution across gender, region, and business type.
**Validates: Requirements 11.4**

### Data Integration Properties

**Property 37: External API integration**
*For any* forecast generation request, the system must attempt to fetch data from configured external APIs (e-NAM, Agmarknet) and include a status indicator of whether external data was successfully integrated.
**Validates: Requirements 12.1**

**Property 38: Optional data source integration**
*For any* forecast where weather data, festival calendars, or local events are available, these data sources must be incorporated, verifiable by checking the data sources list in the response metadata.
**Validates: Requirements 12.2**

**Property 39: Graceful degradation with stale data**
*For any* request when external data sources are unavailable, the system must continue operating with cached data and include a staleness indicator showing the age of the cached data.
**Validates: Requirements 12.3**

**Property 40: Daily data refresh**
*For any* external data source, a refresh operation must be attempted at least once per 24-hour period, with the last refresh timestamp recorded.
**Validates: Requirements 12.4**

**Property 41: Anomalous data rejection**
*For any* external data value that exceeds 3 standard deviations from the historical mean, the value must be flagged as anomalous and excluded from calculations.
**Validates: Requirements 12.5**

### User Experience Properties

**Property 42: Repeated failure help trigger**
*For any* user action that fails more than 3 times consecutively, the system must proactively offer contextual help on the 4th attempt.
**Validates: Requirements 13.3**

### Performance and Scalability Properties

**Property 43: Concurrent user support**
*For any* load test with up to 100,000 concurrent users, the p95 response time for any API endpoint must not exceed 5 seconds.
**Validates: Requirements 14.1**

**Property 44: Auto-scaling responsiveness**
*For any* load increase of 50% or more, compute resources must scale up within 2 minutes, verifiable by monitoring resource allocation timestamps.
**Validates: Requirements 14.2**

**Property 45: Forecast latency bounds**
*For any* forecast request under normal load, p95 latency must be under 2 seconds and p99 latency must be under 5 seconds.
**Validates: Requirements 14.3**

**Property 46: Graceful error degradation**
*For any* system error in a non-critical service, the system must continue serving requests with reduced functionality rather than returning complete failures.
**Validates: Requirements 14.5**

### Reporting Properties

**Property 47: Multi-period report generation**
*For any* performance report request, the system must generate summaries for all three time periods (weekly, monthly, quarterly) with key metrics for each.
**Validates: Requirements 15.1**

**Property 48: Forecast variance analysis**
*For any* performance report, actual outcomes must be compared against forecasts with variance percentages calculated and highlighted when variance exceeds 20%.
**Validates: Requirements 15.2**

**Property 49: Performance change explanation**
*For any* report showing performance change greater than 20% (improvement or decline), the system must identify and explain at least two contributing factors.
**Validates: Requirements 15.3**

**Property 50: Metrics visualization**
*For any* performance report, visualization data for key metrics (revenue, profit margin, inventory turnover) must be included in a format suitable for chart rendering.
**Validates: Requirements 15.4**

**Property 51: Recommendation outcome tracking**
*For any* recommendation that a user explicitly follows, the system must create a tracking record linking the recommendation to subsequent outcomes for effectiveness measurement.
**Validates: Requirements 15.5**


## Error Handling

### Error Classification

**User Errors (4xx):**
- Invalid input data (malformed requests, out-of-range values)
- Authentication/authorization failures
- Rate limit exceeded
- Resource not found

**System Errors (5xx):**
- Service unavailable (temporary)
- External API failures
- Database connection errors
- Model inference failures

**Degraded Mode Errors:**
- Partial data availability
- Stale cache data
- Reduced functionality warnings

### Error Handling Strategies

**Graceful Degradation:**
```typescript
interface ErrorHandlingStrategy {
  // Attempt operation with fallback chain
  executeWithFallback<T>(
    primary: () => Promise<T>,
    fallbacks: Array<() => Promise<T>>,
    degradedMode: () => Promise<T>
  ): Promise<Result<T>>;
}

// Example: Forecast generation with fallbacks
async function generateForecast(request: ForecastRequest): Promise<ForecastResult> {
  return executeWithFallback(
    // Primary: Use latest model with fresh data
    () => latestModel.predict(request),
    
    // Fallback 1: Use previous model version
    [() => previousModel.predict(request),
     
     // Fallback 2: Use regional benchmark model
     () => benchmarkModel.predict(request)],
    
    // Degraded: Return historical average with warning
    () => historicalAverage(request)
  );
}
```

**Retry Logic:**
- Exponential backoff for transient failures
- Maximum 3 retries for external API calls
- Circuit breaker pattern for failing services
- Jitter to prevent thundering herd

**User-Facing Error Messages:**
- Multilingual error messages in user's preferred language
- Clear explanation of what went wrong
- Actionable steps for resolution
- Option to retry or contact support

**Error Response Format:**
```typescript
interface ErrorResponse {
  error: {
    code: string;           // Machine-readable error code
    message: string;        // Human-readable message in user's language
    details?: any;          // Additional context
    retryable: boolean;     // Can user retry?
    suggestedAction?: string; // What should user do?
    supportReference?: string; // Reference for support team
  };
  timestamp: Date;
  requestId: string;
}
```

### Monitoring and Alerting

**Error Metrics:**
- Error rate by service and error type
- Error rate by user segment
- Mean time to recovery (MTTR)
- Error impact (users affected)

**Alert Thresholds:**
- Critical: Error rate >5% for any service
- Warning: Error rate >2% for any service
- Critical: Any service unavailable >5 minutes
- Warning: External API failure rate >10%

**Logging Strategy:**
- Structured logging (JSON format)
- Correlation IDs for request tracing
- PII redaction in logs
- Log levels: DEBUG, INFO, WARN, ERROR, CRITICAL
- Centralized log aggregation (ELK stack)

## Testing Strategy

### Testing Approach

MarketMitra requires a comprehensive testing strategy that combines traditional unit testing with property-based testing to ensure correctness, reliability, and fairness of AI-driven recommendations.

### Dual Testing Approach

**Unit Tests:**
Unit tests verify specific examples, edge cases, and error conditions. They are essential for:
- Testing specific business logic examples
- Verifying integration points between services
- Validating error handling for known failure modes
- Testing edge cases (empty data, boundary values, null inputs)

**Property-Based Tests:**
Property-based tests verify universal properties across all inputs through randomization. They are essential for:
- Validating correctness properties that must hold for all valid inputs
- Discovering edge cases through random input generation
- Testing AI model behavior across diverse scenarios
- Ensuring fairness and consistency across user segments

**Balance:** Avoid writing too many unit tests for scenarios that property-based tests can cover. Focus unit tests on specific examples and integration points, while property tests handle comprehensive input coverage.

### Property-Based Testing Configuration

**Framework Selection:**
- **Python services**: Use Hypothesis library
- **TypeScript/JavaScript services**: Use fast-check library
- **Integration tests**: Use Hypothesis or fast-check depending on test harness

**Test Configuration:**
- Minimum 100 iterations per property test (due to randomization)
- Configurable seed for reproducibility
- Shrinking enabled to find minimal failing examples
- Timeout: 30 seconds per property test

**Property Test Tagging:**
Each property-based test must include a comment tag referencing the design document property:

```python
# Feature: market-mitra-decision-support, Property 1: Multi-horizon forecast generation
@given(business_profile=business_profiles(), product=products())
def test_forecast_returns_all_horizons(business_profile, product):
    forecast = forecast_service.generate_forecast(
        user_id=business_profile.user_id,
        product_id=product.id,
        horizons=[7, 14, 30]
    )
    
    assert len(forecast.horizons) == 3
    assert 7 in [h.days for h in forecast.horizons]
    assert 14 in [h.days for h in forecast.horizons]
    assert 30 in [h.days for h in forecast.horizons]
    
    for horizon in forecast.horizons:
        assert len(horizon.daily_forecasts) == horizon.days
```

```typescript
// Feature: market-mitra-decision-support, Property 5: Three-tier pricing structure
fc.assert(
  fc.property(
    pricingRequestArbitrary(),
    async (request) => {
      const recommendation = await pricingService.recommendPrice(request);
      
      expect(recommendation.recommendations).toHaveLength(3);
      
      const types = recommendation.recommendations.map(r => r.type);
      expect(types).toContain('minimum');
      expect(types).toContain('optimal');
      expect(types).toContain('maximum');
      
      for (const rec of recommendation.recommendations) {
        expect(rec.expectedVolume).toBeGreaterThanOrEqual(0);
        expect(rec.expectedRevenue).toBeGreaterThanOrEqual(0);
        expect(rec.expectedProfit).toBeDefined();
      }
    }
  ),
  { numRuns: 100 }
);
```

### Test Data Generators

**Custom Generators for Domain Objects:**

```python
from hypothesis import strategies as st

@st.composite
def business_profiles(draw):
    return BusinessProfile(
        user_id=draw(st.uuids()),
        business_name=draw(st.text(min_size=1, max_size=100)),
        business_type=draw(st.sampled_from(['grocery', 'dairy', 'handicraft'])),
        location=draw(locations()),
        products=draw(st.lists(products(), min_size=1, max_size=10)),
        preferred_language=draw(st.sampled_from(['hi', 'ta', 'te', 'bn', 'mr', 'gu']))
    )

@st.composite
def products(draw):
    return Product(
        id=draw(st.uuids()),
        name=draw(st.text(min_size=1, max_size=50)),
        category=draw(st.sampled_from(['food', 'dairy', 'handicraft', 'agriculture'])),
        cost_price=draw(st.floats(min_value=1.0, max_value=10000.0)),
        selling_price=draw(st.floats(min_value=1.0, max_value=15000.0)),
        perishable=draw(st.booleans()),
        shelf_life_days=draw(st.integers(min_value=1, max_value=365)) if draw(st.booleans()) else None
    )
```

### Test Coverage by Component

**Forecast Service:**
- Unit tests: 15-20 tests covering specific scenarios (seasonal products, new products, data gaps)
- Property tests: 8 tests covering Properties 1-4
- Integration tests: 5 tests covering external API integration

**Pricing Service:**
- Unit tests: 12-15 tests covering specific pricing scenarios
- Property tests: 3 tests covering Properties 5-7
- Integration tests: 3 tests covering competitor price fetching

**Inventory Service:**
- Unit tests: 15-20 tests covering perishability, constraints, edge cases
- Property tests: 4 tests covering Properties 8-11
- Integration tests: 3 tests covering forecast integration

**PnL Simulator:**
- Unit tests: 10-12 tests covering specific scenarios
- Property tests: 4 tests covering Properties 12-15
- Integration tests: 2 tests covering multi-service integration

**Voice Interface:**
- Unit tests: 20-25 tests covering language-specific scenarios
- Property tests: 3 tests covering Properties 16-18
- Integration tests: 6 tests (one per language)

**Explainability Service:**
- Unit tests: 10-12 tests covering explanation generation
- Property tests: 3 tests covering Properties 19-21
- Integration tests: 4 tests covering SHAP/LIME integration

**Business Profile Service:**
- Unit tests: 15-18 tests covering CRUD operations
- Property tests: 2 tests covering Properties 22-23
- Integration tests: 3 tests covering cascade updates

### AI Model Testing

**Model Validation:**
- Holdout test set (20% of data) for accuracy measurement
- Cross-validation (5-fold) during training
- Temporal validation (train on past, test on future)
- Fairness testing across demographic segments

**Model Performance Tests:**
```python
# Feature: market-mitra-decision-support, Property 30: Forecast accuracy tracking
def test_forecast_accuracy_within_bounds():
    """Test that forecast MAPE is within acceptable bounds"""
    test_data = load_test_dataset()
    
    for product_category in test_data.categories:
        forecasts = generate_forecasts_for_category(product_category)
        actuals = get_actual_outcomes(product_category)
        
        mape = calculate_mape(forecasts, actuals)
        
        # MAPE should be < 30% for 30-day forecasts
        assert mape < 0.30, f"MAPE {mape} exceeds threshold for {product_category}"
```

**Fairness Testing:**
```python
# Feature: market-mitra-decision-support, Property 36: Training data diversity
@given(training_dataset=training_datasets())
def test_training_data_demographic_balance(training_dataset):
    """Test that training data represents diverse demographics"""
    demographics = analyze_demographics(training_dataset)
    target_distribution = get_target_population_distribution()
    
    for demographic_group in demographics.keys():
        actual_pct = demographics[demographic_group]
        target_pct = target_distribution[demographic_group]
        
        # Within 20% of target distribution
        assert abs(actual_pct - target_pct) <= 0.20
```

### Integration Testing

**Service Integration Tests:**
- Test communication between services
- Test message queue processing
- Test event-driven workflows
- Test external API integration

**End-to-End Tests:**
- User journey tests (onboarding → forecast → pricing → decision)
- Multi-language user flows
- Offline/online transition scenarios
- Error recovery scenarios

### Performance Testing

**Load Testing:**
- Gradual ramp-up to 100,000 concurrent users
- Sustained load testing (24 hours)
- Spike testing (sudden 10x load increase)
- Stress testing (find breaking point)

**Tools:**
- k6 or Locust for load generation
- Prometheus + Grafana for monitoring
- Custom scripts for AI model latency testing

### Security Testing

**Automated Security Scans:**
- SAST (Static Application Security Testing) in CI/CD
- DAST (Dynamic Application Security Testing) on staging
- Dependency vulnerability scanning
- Container image scanning

**Penetration Testing:**
- Quarterly penetration tests by external security firm
- Focus areas: authentication, data encryption, API security

### Continuous Integration/Deployment

**CI Pipeline:**
1. Code commit triggers build
2. Run linters and formatters
3. Run unit tests (parallel execution)
4. Run property-based tests (parallel execution)
5. Run integration tests
6. Build container images
7. Run security scans
8. Deploy to staging

**CD Pipeline:**
1. Automated deployment to staging
2. Run smoke tests
3. Run end-to-end tests
4. Manual approval gate
5. Blue-green deployment to production
6. Monitor error rates and performance
7. Automatic rollback if error rate >2%

**Test Execution Time Targets:**
- Unit tests: < 5 minutes
- Property-based tests: < 15 minutes
- Integration tests: < 10 minutes
- End-to-end tests: < 20 minutes
- Total CI pipeline: < 30 minutes

### Test Environment Management

**Environments:**
- **Development**: Local development with mocked external services
- **Staging**: Production-like environment with test data
- **Production**: Live environment with real user data

**Test Data Management:**
- Synthetic data generation for testing
- Anonymized production data for staging (with user consent)
- Separate databases for each environment
- Automated data refresh for staging (weekly)

### Monitoring and Observability

**Production Monitoring:**
- Real-time error tracking (Sentry or similar)
- Performance monitoring (APM tools)
- AI model performance dashboards
- User experience metrics (response times, success rates)

**Alerting:**
- PagerDuty or similar for critical alerts
- Slack notifications for warnings
- Weekly summary reports for stakeholders

### Documentation

**Test Documentation:**
- Property-based test catalog (mapping properties to tests)
- Test data generator documentation
- Integration test scenarios
- Performance test results and baselines

**Developer Documentation:**
- Testing guidelines and best practices
- How to write property-based tests
- How to run tests locally
- How to debug failing tests

## Deployment Architecture

### Multi-Region Deployment

**Primary Region:** Mumbai (Asia South)
**Secondary Region:** Bangalore (Asia South 2)

**Traffic Distribution:**
- 70% traffic to primary region
- 30% traffic to secondary region
- Automatic failover if primary region unavailable

### Infrastructure Components

**Compute:**
- Kubernetes cluster (3 nodes minimum per region)
- Auto-scaling: 3-20 nodes based on load
- Node types: Mixed (CPU-optimized for services, GPU-optimized for ML)

**Storage:**
- PostgreSQL: Multi-AZ deployment with read replicas
- TimescaleDB: Separate cluster for time-series data
- Redis: Cluster mode with 3 master nodes
- S3: Multi-region replication for model artifacts

**Networking:**
- Load balancer: Application Load Balancer (ALB)
- CDN: CloudFlare for static assets and API caching
- VPC: Private subnets for databases, public subnets for load balancers
- VPN: For administrative access

### Disaster Recovery

**Backup Strategy:**
- Database backups: Every 6 hours, retained for 30 days
- Model artifacts: Versioned in S3 with lifecycle policies
- Configuration: Stored in Git, deployed via GitOps

**Recovery Objectives:**
- RTO (Recovery Time Objective): 1 hour
- RPO (Recovery Point Objective): 6 hours

**Failover Procedures:**
- Automated failover for database (Multi-AZ)
- Manual failover for region (requires DNS update)
- Runbook for disaster recovery scenarios

## Future Enhancements

### Phase 2 Features (Months 7-12)

1. **WhatsApp Bot Integration**: Native WhatsApp interface for voice and text
2. **Supplier Network**: Connect entrepreneurs with suppliers for better pricing
3. **Peer Benchmarking**: Compare performance with similar businesses
4. **Credit Scoring**: Help entrepreneurs access microfinance
5. **Seasonal Planning**: Long-term planning tools for seasonal businesses

### Phase 3 Features (Months 13-24)

1. **Cooperative Features**: Tools for business cooperatives
2. **Export Market Intelligence**: Insights for export opportunities
3. **Sustainability Metrics**: Track and improve environmental impact
4. **Advanced Analytics**: Predictive analytics for market trends
5. **Mobile App**: Native mobile apps for iOS and Android

### Research Areas

1. **Federated Learning**: Train models without centralizing sensitive data
2. **Edge AI**: Run models on-device for offline functionality
3. **Causal Inference**: Move beyond correlation to understand causation
4. **Multi-Agent Systems**: Simulate market dynamics for better predictions
5. **Explainable Deep Learning**: Improve explanation quality for complex models
