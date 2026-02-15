# AdLily: Technical Design Document

**Version:** 1.0  
**Date:** February 15, 2026  
**Status:** Draft  

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Technology Stack](#technology-stack)
3. [Data Architecture](#data-architecture)
4. [AI/ML Architecture](#aiml-architecture)
5. [API Design](#api-design)
6. [Infrastructure](#infrastructure)
7. [Security Architecture](#security-architecture)
8. [Scalability & Performance](#scalability--performance)

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                │
│  Web App (React) • Mobile App (React Native - Future)               │
└─────────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                          API GATEWAY                                 │
│  AWS API Gateway • Authentication • Rate Limiting • Routing          │
└─────────────────────────────────────────────────────────────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                       APPLICATION LAYER                              │
├──────────────────┬──────────────────┬──────────────────┬────────────┤
│  User Service    │  Campaign Service│  Insight Service │ Gen Service│
│  (FastAPI)       │  (FastAPI)       │  (FastAPI)       │ (FastAPI)  │
└──────────────────┴──────────────────┴──────────────────┴────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                       PROCESSING LAYER                               │
├──────────────────┬──────────────────┬──────────────────┬────────────┤
│  Data Ingestion  │  AI/ML Pipeline  │  Performance     │ Optimization│
│  Workers (SQS)   │  (Bedrock/Sagemaker)│ Tracking      │ Engine     │
└──────────────────┴──────────────────┴──────────────────┴────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                  │
├──────────────────┬──────────────────┬──────────────────┬────────────┤
│  PostgreSQL      │  S3 (Assets)     │  Redis (Cache)   │ OpenSearch │
│  (RDS)           │  (Videos/Images) │  (Sessions)      │ (Analytics)│
└──────────────────┴──────────────────┴──────────────────┴────────────┘
                                  ↓
┌─────────────────────────────────────────────────────────────────────┐
│                      EXTERNAL INTEGRATIONS                           │
├──────────────────┬──────────────────┬──────────────────┬────────────┤
│  Review APIs     │  Support APIs    │  Ad Platform APIs│ AI Services│
│  (Amazon, G2)    │  (Zendesk, etc.) │  (Meta, Google)  │ (Bedrock)  │
└──────────────────┴──────────────────┴──────────────────┴────────────┘
```

### Service Architecture (Microservices)

#### 1. User Service
**Responsibilities:**
- User authentication and authorization
- Account management
- Subscription and billing
- Team management
- Permissions and RBAC

**Tech Stack:**
- FastAPI (Python)
- PostgreSQL (user data)
- Redis (sessions)
- Stripe API (billing)

#### 2. Insight Service
**Responsibilities:**
- Data ingestion from external sources
- Sentiment analysis
- Insight extraction
- Competitor tracking
- Viral hook detection

**Tech Stack:**
- FastAPI (Python)
- PostgreSQL (insights data)
- AWS Bedrock (Claude for analysis)
- SQS (async processing)
- Lambda (scheduled jobs)

#### 3. Campaign Service
**Responsibilities:**
- Campaign creation and management
- Ad concept generation
- Ad scoring
- Recommendations
- Campaign analytics

**Tech Stack:**
- FastAPI (Python)
- PostgreSQL (campaign data)
- Redis (caching)
- AWS Bedrock (Claude for concepts)

#### 4. Generation Service
**Responsibilities:**
- Video generation
- Image generation
- Copy generation
- Asset management
- Brand style management

**Tech Stack:**
- FastAPI (Python)
- AWS Bedrock (Stable Diffusion, Titan)
- S3 (asset storage)
- SQS (generation queue)
- ECS (GPU instances for generation)

#### 5. Performance Service
**Responsibilities:**
- Ad platform integration
- Performance data sync
- Analytics and reporting
- ROI calculation
- Attribution modeling

**Tech Stack:**
- FastAPI (Python)
- PostgreSQL (performance data)
- OpenSearch (analytics)
- Redis (real-time metrics)
- Lambda (scheduled syncs)

#### 6. Optimization Service
**Responsibilities:**
- Performance monitoring
- Recommendation generation
- Automated optimization
- Creative fatigue detection
- Model retraining

**Tech Stack:**
- FastAPI (Python)
- PostgreSQL (optimization data)
- SageMaker (ML models)
- SQS (async processing)
- Lambda (scheduled jobs)

---

## Technology Stack

### Frontend

**Web Application:**
- **Framework:** React 18 with TypeScript
- **State Management:** Redux Toolkit + RTK Query
- **UI Library:** Material-UI (MUI) v5
- **Charts:** Recharts, D3.js
- **Video Player:** Video.js
- **Forms:** React Hook Form + Zod validation
- **Routing:** React Router v6
- **Build Tool:** Vite
- **Testing:** Jest, React Testing Library, Playwright

**Design System:**
- Custom component library
- Figma design tokens
- Storybook for component documentation

### Backend

**API Layer:**
- **Framework:** FastAPI (Python 3.11+)
- **Async:** asyncio, aiohttp
- **Validation:** Pydantic v2
- **ORM:** SQLAlchemy 2.0 (async)
- **Migrations:** Alembic
- **API Docs:** OpenAPI/Swagger (auto-generated)

**Background Workers:**
- **Queue:** AWS SQS
- **Worker Framework:** Celery (alternative: custom async workers)
- **Scheduler:** AWS EventBridge (cron jobs)

### Database

**Primary Database:**
- **Type:** PostgreSQL 15 (AWS RDS)
- **Features:** JSONB, Full-text search, Partitioning
- **Replication:** Multi-AZ, Read replicas
- **Backup:** Automated daily snapshots

**Cache:**
- **Type:** Redis 7 (AWS ElastiCache)
- **Use Cases:** Sessions, API cache, real-time metrics
- **Persistence:** AOF + RDB

**Analytics:**
- **Type:** OpenSearch (AWS)
- **Use Cases:** Performance analytics, log aggregation
- **Retention:** 90 days hot, 1 year warm

**Object Storage:**
- **Type:** AWS S3
- **Use Cases:** Videos, images, assets, backups
- **Lifecycle:** Intelligent-Tiering, Glacier for archives

### AI/ML

**LLM Services:**
- **Primary:** AWS Bedrock (Claude 3.5 Sonnet)
- **Use Cases:** Insight extraction, copy generation, concept generation
- **Fallback:** OpenAI GPT-4 (if Bedrock unavailable)

**Image Generation:**
- **Primary:** AWS Bedrock (Stable Diffusion XL, Titan Image)
- **Fallback:** Replicate API (Stable Diffusion)

**Video Generation:**
- **Primary:** Custom pipeline (Stable Diffusion Video + compositing)
- **Fallback:** RunwayML Gen-2 API

**ML Models:**
- **Training:** AWS SageMaker
- **Inference:** SageMaker Endpoints (real-time), Batch Transform (batch)
- **Model Registry:** SageMaker Model Registry
- **Monitoring:** SageMaker Model Monitor

### Infrastructure

**Cloud Provider:** AWS

**Compute:**
- **API Services:** ECS Fargate (containerized)
- **Background Workers:** ECS Fargate + EC2 (GPU for generation)
- **Serverless:** Lambda (scheduled jobs, webhooks)

**Networking:**
- **Load Balancer:** Application Load Balancer (ALB)
- **CDN:** CloudFront (static assets, videos)
- **DNS:** Route 53
- **VPC:** Private subnets for services, public for ALB

**Monitoring:**
- **Metrics:** CloudWatch, Prometheus
- **Logging:** CloudWatch Logs, OpenSearch
- **Tracing:** AWS X-Ray
- **Alerting:** CloudWatch Alarms, PagerDuty
- **APM:** Datadog (optional)

**CI/CD:**
- **Version Control:** GitHub
- **CI/CD:** GitHub Actions
- **Container Registry:** AWS ECR
- **IaC:** Terraform
- **Secrets:** AWS Secrets Manager

### External Integrations

**Review Platforms:**
- Amazon Product Advertising API
- G2 API
- Trustpilot API
- Yelp Fusion API
- App Store Connect API
- Google Play Developer API

**Support Platforms:**
- Zendesk API
- Intercom API
- Freshdesk API
- Salesforce Service Cloud API

**Ad Platforms:**
- Meta Marketing API
- Google Ads API
- TikTok Marketing API
- LinkedIn Marketing API

**Payment:**
- Stripe API (subscriptions, billing)

**Communication:**
- SendGrid (transactional email)
- Twilio (SMS - future)
- Slack API (notifications - future)

---

## Data Architecture

### Database Schema (PostgreSQL)

#### Core Tables

**users**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255),
    name VARCHAR(255),
    company VARCHAR(255),
    role VARCHAR(50) DEFAULT 'user',
    plan VARCHAR(50) DEFAULT 'starter',
    stripe_customer_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    last_login_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_stripe ON users(stripe_customer_id);
```

**brands**
```sql
CREATE TABLE brands (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    domain VARCHAR(255),
    industry VARCHAR(100),
    brand_voice JSONB, -- {tone, style, guidelines}
    brand_assets JSONB, -- {logo_url, colors, fonts}
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_brands_user ON brands(user_id);
```

**data_sources**
```sql
CREATE TABLE data_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID REFERENCES brands(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL, -- 'review', 'support', 'ad_platform'
    platform VARCHAR(100) NOT NULL, -- 'amazon', 'zendesk', 'meta'
    credentials JSONB, -- encrypted API keys, tokens
    config JSONB, -- sync settings, filters
    last_sync_at TIMESTAMP,
    sync_status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_data_sources_brand ON data_sources(brand_id);
CREATE INDEX idx_data_sources_type ON data_sources(type);
```

**reviews**
```sql
CREATE TABLE reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID REFERENCES brands(id) ON DELETE CASCADE,
    source_id UUID REFERENCES data_sources(id),
    external_id VARCHAR(255), -- ID from source platform
    platform VARCHAR(100) NOT NULL,
    rating DECIMAL(2,1),
    title TEXT,
    content TEXT NOT NULL,
    author_name VARCHAR(255),
    verified_purchase BOOLEAN,
    helpful_votes INTEGER DEFAULT 0,
    review_date DATE,
    sentiment_score DECIMAL(3,2), -- -5.00 to +5.00
    emotions JSONB, -- {joy: 0.8, frustration: 0.2}
    extracted_insights JSONB, -- {pain_points: [], benefits: []}
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reviews_brand ON reviews(brand_id);
CREATE INDEX idx_reviews_sentiment ON reviews(sentiment_score);
CREATE INDEX idx_reviews_date ON reviews(review_date);
CREATE INDEX idx_reviews_platform ON reviews(platform);
```

**support_tickets**
```sql
CREATE TABLE support_tickets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID REFERENCES brands(id) ON DELETE CASCADE,
    source_id UUID REFERENCES data_sources(id),
    external_id VARCHAR(255),
    platform VARCHAR(100) NOT NULL,
    subject TEXT,
    content TEXT NOT NULL,
    category VARCHAR(100),
    priority VARCHAR(50),
    status VARCHAR(50),
    resolution_time INTEGER, -- minutes
    customer_satisfaction INTEGER, -- 1-5
    sentiment_score DECIMAL(3,2),
    extracted_insights JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    resolved_at TIMESTAMP
);

CREATE INDEX idx_tickets_brand ON support_tickets(brand_id);
CREATE INDEX idx_tickets_category ON support_tickets(category);
CREATE INDEX idx_tickets_status ON support_tickets(status);
```

**insights**
```sql
CREATE TABLE insights (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID REFERENCES brands(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL, -- 'pain_point', 'benefit', 'objection', 'competitor_gap', 'viral_hook'
    content TEXT NOT NULL,
    source_type VARCHAR(50), -- 'review', 'support', 'competitor', 'social'
    frequency INTEGER DEFAULT 1,
    sentiment_score DECIMAL(3,2),
    impact_score DECIMAL(5,2), -- calculated score
    metadata JSONB, -- {examples: [], sources: []}
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_insights_brand ON insights(brand_id);
CREATE INDEX idx_insights_type ON insights(type);
CREATE INDEX idx_insights_impact ON insights(impact_score DESC);
```

**campaigns**
```sql
CREATE TABLE campaigns (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    brand_id UUID REFERENCES brands(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    objective VARCHAR(50), -- 'awareness', 'consideration', 'conversion'
    status VARCHAR(50) DEFAULT 'draft', -- 'draft', 'active', 'paused', 'completed'
    budget_daily DECIMAL(10,2),
    budget_lifetime DECIMAL(10,2),
    start_date DATE,
    end_date DATE,
    platforms JSONB, -- ['meta', 'google']
    target_audience JSONB,
    selected_insights JSONB, -- [insight_ids]
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_campaigns_brand ON campaigns(brand_id);
CREATE INDEX idx_campaigns_status ON campaigns(status);
```

**ad_concepts**
```sql
CREATE TABLE ad_concepts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
    hook TEXT NOT NULL,
    message TEXT NOT NULL,
    cta TEXT NOT NULL,
    predicted_score DECIMAL(5,2), -- 0-100
    score_breakdown JSONB, -- {relevance: 85, emotional_impact: 90, ...}
    confidence_interval JSONB, -- {lower: 82, upper: 88}
    reasoning TEXT,
    predicted_metrics JSONB, -- {ctr: [3.2, 3.8], roas: [2.8, 3.4]}
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_concepts_campaign ON ad_concepts(campaign_id);
CREATE INDEX idx_concepts_score ON ad_concepts(predicted_score DESC);
```

**ads**
```sql
CREATE TABLE ads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    concept_id UUID REFERENCES ad_concepts(id),
    campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
    format VARCHAR(50) NOT NULL, -- 'video', 'image', 'carousel'
    platform VARCHAR(50) NOT NULL,
    headline TEXT,
    body_copy TEXT,
    cta TEXT,
    asset_urls JSONB, -- {video: 's3://...', thumbnail: 's3://...'}
    platform_ad_id VARCHAR(255), -- ID from ad platform
    status VARCHAR(50) DEFAULT 'draft',
    created_at TIMESTAMP DEFAULT NOW(),
    published_at TIMESTAMP
);

CREATE INDEX idx_ads_campaign ON ads(campaign_id);
CREATE INDEX idx_ads_concept ON ads(concept_id);
CREATE INDEX idx_ads_platform ON ads(platform, platform_ad_id);
```

**ad_performance**
```sql
CREATE TABLE ad_performance (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ad_id UUID REFERENCES ads(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    platform VARCHAR(50) NOT NULL,
    impressions INTEGER DEFAULT 0,
    clicks INTEGER DEFAULT 0,
    conversions INTEGER DEFAULT 0,
    spend DECIMAL(10,2) DEFAULT 0,
    revenue DECIMAL(10,2) DEFAULT 0,
    ctr DECIMAL(5,4), -- click-through rate
    cpc DECIMAL(10,2), -- cost per click
    cpa DECIMAL(10,2), -- cost per acquisition
    roas DECIMAL(10,2), -- return on ad spend
    engagement_rate DECIMAL(5,4),
    video_completion_rate DECIMAL(5,4),
    metadata JSONB, -- platform-specific metrics
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_performance_ad ON ad_performance(ad_id);
CREATE INDEX idx_performance_date ON ad_performance(date);
CREATE UNIQUE INDEX idx_performance_unique ON ad_performance(ad_id, date, platform);
```


**recommendations**
```sql
CREATE TABLE recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES campaigns(id),
    ad_id UUID REFERENCES ads(id),
    type VARCHAR(50) NOT NULL, -- 'pause', 'scale', 'refresh', 'reallocate'
    title TEXT NOT NULL,
    description TEXT,
    reasoning TEXT,
    confidence DECIMAL(5,2), -- 0-100
    priority VARCHAR(20), -- 'low', 'medium', 'high', 'critical'
    suggested_action JSONB,
    status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'approved', 'rejected', 'executed'
    created_at TIMESTAMP DEFAULT NOW(),
    executed_at TIMESTAMP
);

CREATE INDEX idx_recommendations_campaign ON recommendations(campaign_id);
CREATE INDEX idx_recommendations_status ON recommendations(status);
CREATE INDEX idx_recommendations_priority ON recommendations(priority);
```

### Data Partitioning Strategy

**Time-Series Data (Partitioned by Month):**
- `ad_performance` - Partition by date (monthly)
- `reviews` - Partition by review_date (monthly)
- `support_tickets` - Partition by created_at (monthly)

**Benefits:**
- Faster queries (scan only relevant partitions)
- Easier archival (drop old partitions)
- Better maintenance (vacuum, analyze per partition)

**Example:**
```sql
CREATE TABLE ad_performance (
    -- columns as above
) PARTITION BY RANGE (date);

CREATE TABLE ad_performance_2026_01 PARTITION OF ad_performance
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE ad_performance_2026_02 PARTITION OF ad_performance
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
-- etc.
```

### Data Retention Policy

| Table | Retention | Archive Strategy |
|-------|-----------|------------------|
| users | Indefinite | N/A |
| brands | Indefinite | N/A |
| campaigns | 2 years | Move to S3 (Parquet) |
| ads | 2 years | Move to S3 |
| ad_performance | 2 years hot, 5 years warm | OpenSearch (90 days), S3 (long-term) |
| reviews | 2 years | Move to S3 |
| support_tickets | 1 year | Move to S3 |
| insights | Indefinite | Aggregate old data |
| recommendations | 6 months | Delete after execution |

---

## AI/ML Architecture

### 1. Insight Extraction Pipeline

**Architecture:**
```
Reviews/Tickets → Preprocessing → LLM Analysis → Insight Extraction → Scoring → Storage
```

**Components:**

**1.1 Preprocessing:**
- Text cleaning (remove HTML, special chars)
- Language detection
- Translation (if non-English)
- PII redaction
- Deduplication

**1.2 LLM Analysis (AWS Bedrock - Claude):**
```python
# Prompt template for insight extraction
INSIGHT_EXTRACTION_PROMPT = """
Analyze the following customer reviews and extract:
1. Top pain points (problems customers face)
2. Top benefits (outcomes customers love)
3. Common objections (reasons for hesitation)
4. Emotional triggers (what makes them feel strongly)

Reviews:
{reviews_text}

Output format (JSON):
{
  "pain_points": [{"text": "...", "frequency": N, "severity": 1-5}],
  "benefits": [{"text": "...", "frequency": N, "sentiment": 1-5}],
  "objections": [{"text": "...", "frequency": N}],
  "emotional_triggers": [{"emotion": "...", "trigger": "...", "intensity": 1-5}]
}
"""
```

**1.3 Insight Scoring:**
```python
def calculate_insight_impact_score(insight):
    """
    Impact Score = (Frequency × Sentiment × Recency) / 100
    
    - Frequency: How often mentioned (1-100)
    - Sentiment: Intensity of sentiment (-5 to +5, normalized to 0-100)
    - Recency: Time decay factor (1.0 for recent, 0.5 for 6 months old)
    """
    frequency_score = min(insight.frequency, 100)
    sentiment_score = (insight.sentiment_score + 5) * 10  # -5 to +5 → 0 to 100
    recency_factor = calculate_recency_factor(insight.created_at)
    
    impact_score = (frequency_score * sentiment_score * recency_factor) / 100
    return impact_score
```

**1.4 Batch Processing:**
- Process reviews in batches of 100
- Parallel processing (10 concurrent batches)
- Rate limiting (respect API limits)
- Retry logic (exponential backoff)

### 2. Ad Scoring Model

**Model Type:** Ensemble (XGBoost + Neural Network)

**Features:**

**Text Features:**
- Headline embeddings (Sentence-BERT)
- Body copy embeddings
- CTA embeddings
- Customer language similarity (cosine similarity with reviews)
- Readability scores (Flesch-Kincaid)
- Sentiment scores
- Emotional intensity

**Visual Features (for video/image ads):**
- Image embeddings (CLIP)
- Color palette analysis
- Face detection (presence, emotion)
- Text overlay analysis
- Brand consistency score

**Metadata Features:**
- Platform (Meta, Google, TikTok)
- Format (video, image, carousel)
- Duration (for video)
- Target audience demographics
- Campaign objective
- Budget level

**Insight Features:**
- Pain point alignment score
- Benefit highlighting score
- Objection addressing score
- Competitor differentiation score
- Viral hook usage (binary)

**Historical Features:**
- Brand's past performance (avg ROAS)
- Similar ad performance (k-NN)
- Industry benchmarks
- Platform-specific patterns

**Model Architecture:**

```python
# Ensemble model
class AdScoringEnsemble:
    def __init__(self):
        self.xgboost_model = XGBRegressor(...)
        self.neural_net = NeuralNetRegressor(...)
        self.weights = [0.6, 0.4]  # XGBoost, NN
    
    def predict(self, features):
        xgb_pred = self.xgboost_model.predict(features)
        nn_pred = self.neural_net.predict(features)
        
        ensemble_pred = (
            self.weights[0] * xgb_pred + 
            self.weights[1] * nn_pred
        )
        
        # Confidence interval (bootstrap)
        confidence_interval = self.calculate_confidence_interval(features)
        
        return {
            'score': ensemble_pred,
            'confidence_interval': confidence_interval,
            'feature_importance': self.get_feature_importance()
        }
```

**Training:**
- Training data: Historical ad performance (internal + external datasets)
- Target variable: Actual ROAS (normalized 0-100)
- Train/validation/test split: 70/15/15
- Cross-validation: 5-fold
- Hyperparameter tuning: Optuna
- Retraining frequency: Weekly (incremental)

**Evaluation Metrics:**
- Correlation with actual ROAS (target: r > 0.75)
- Mean Absolute Error (MAE)
- Calibration (predicted vs. actual distribution)
- Feature importance (SHAP values)

### 3. Video Generation Pipeline

**Architecture:**
```
Concept → Script → Scene Plan → Visual Gen → Compositing → Audio → Export
```

**Components:**

**3.1 Script Generation (Claude):**
```python
SCRIPT_GENERATION_PROMPT = """
Create a {duration}-second video ad script for:

Product: {product_name}
Hook: {hook}
Key Message: {message}
CTA: {cta}
Customer Insights: {insights}
Brand Voice: {brand_voice}

Output format:
{
  "scenes": [
    {
      "duration": 3,
      "visual": "...",
      "text_overlay": "...",
      "voiceover": "..."
    }
  ],
  "music_style": "...",
  "pacing": "fast/medium/slow"
}
"""
```

**3.2 Visual Generation:**

**Option A: Template-Based (MVP)**
- Pre-designed templates (After Effects, Remotion)
- Dynamic text and image insertion
- Fast generation (<30 seconds)
- Limited creativity

**Option B: AI-Native (Phase 2)**
- Stable Diffusion Video (AWS Bedrock)
- Text-to-video generation
- Product compositing (overlay product images)
- Slower generation (2-3 minutes)
- Higher creativity

**3.3 Compositing:**
- Product image overlay
- Brand logo placement
- Text overlays (captions, CTAs)
- Color grading (brand colors)
- Transitions and effects

**3.4 Audio:**
- Background music (licensed library)
- Voiceover (Amazon Polly or licensed)
- Sound effects
- Audio mixing and mastering

**3.5 Export:**
- Multiple aspect ratios (9:16, 1:1, 16:9, 4:5)
- Platform-specific specs (bitrate, codec)
- Thumbnail generation
- Caption file (SRT)

**Implementation (Remotion for MVP):**
```typescript
// Remotion component for video generation
export const AdVideo: React.FC<{
  script: Script;
  brandAssets: BrandAssets;
  duration: number;
}> = ({ script, brandAssets, duration }) => {
  return (
    <AbsoluteFill>
      {script.scenes.map((scene, index) => (
        <Sequence
          key={index}
          from={scene.startFrame}
          durationInFrames={scene.durationFrames}
        >
          <Scene
            visual={scene.visual}
            textOverlay={scene.textOverlay}
            brandAssets={brandAssets}
          />
        </Sequence>
      ))}
      <Audio src={script.musicUrl} />
    </AbsoluteFill>
  );
};
```

### 4. Optimization Engine

**Architecture:**
```
Performance Data → Analysis → Recommendation Gen → Execution (if auto)
```

**Components:**

**4.1 Performance Monitoring:**
- Real-time sync (hourly from ad platforms)
- Statistical significance testing
- Anomaly detection (sudden drops/spikes)
- Creative fatigue detection

**4.2 Recommendation Generation:**

**Algorithm:**
```python
def generate_recommendations(campaign):
    recommendations = []
    
    # 1. Identify underperformers
    for ad in campaign.ads:
        if is_underperforming(ad):
            rec = {
                'type': 'pause',
                'ad_id': ad.id,
                'reasoning': f'CPA ${ad.cpa} exceeds target ${campaign.target_cpa}',
                'confidence': calculate_confidence(ad),
                'priority': 'high' if ad.spend > 1000 else 'medium'
            }
            recommendations.append(rec)
    
    # 2. Identify high performers
    for ad in campaign.ads:
        if is_high_performer(ad):
            rec = {
                'type': 'scale',
                'ad_id': ad.id,
                'reasoning': f'ROAS {ad.roas}x exceeds average {campaign.avg_roas}x by 45%',
                'suggested_action': {'budget_multiplier': 2.5},
                'confidence': calculate_confidence(ad),
                'priority': 'high'
            }
            recommendations.append(rec)
    
    # 3. Detect creative fatigue
    for ad in campaign.ads:
        if is_fatigued(ad):
            rec = {
                'type': 'refresh',
                'ad_id': ad.id,
                'reasoning': f'CTR dropped 35% over 5 days, frequency at 3.2',
                'suggested_action': {'generate_variants': True},
                'confidence': 87,
                'priority': 'medium'
            }
            recommendations.append(rec)
    
    return recommendations

def is_underperforming(ad):
    # Statistical significance test
    if ad.conversions < 30:  # Minimum sample size
        return False
    
    # Compare to campaign average
    if ad.cpa > campaign.target_cpa * 1.3:  # 30% worse
        return True
    
    # Compare to predicted performance
    if ad.actual_roas < ad.predicted_roas * 0.7:  # 30% below prediction
        return True
    
    return False
```

**4.3 Automated Execution:**
- User-defined rules (e.g., "auto-pause if CPA > $80")
- Guardrails (max budget change per day)
- Approval workflow (for high-impact actions)
- Rollback capability

### 5. Continuous Learning

**Model Retraining Pipeline:**
```
New Performance Data → Feature Engineering → Model Training → Validation → Deployment
```

**Schedule:**
- Ad scoring model: Weekly retraining
- Insight extraction: Monthly refinement
- Optimization rules: Continuous (A/B testing)

**A/B Testing Framework:**
- Test new model versions against production
- 10% traffic to new model
- Monitor performance metrics
- Gradual rollout if successful

---

## API Design

### RESTful API Endpoints

**Base URL:** `https://api.adlily.com/v1`

**Authentication:** Bearer token (JWT)

### User & Auth Endpoints

```
POST   /auth/register          # Register new user
POST   /auth/login             # Login
POST   /auth/logout            # Logout
POST   /auth/refresh           # Refresh token
GET    /auth/me                # Get current user
PUT    /auth/me                # Update current user
POST   /auth/reset-password    # Request password reset
```

### Brand Endpoints

```
GET    /brands                 # List user's brands
POST   /brands                 # Create brand
GET    /brands/:id             # Get brand details
PUT    /brands/:id             # Update brand
DELETE /brands/:id             # Delete brand
POST   /brands/:id/assets      # Upload brand assets
```

### Data Source Endpoints

```
GET    /brands/:id/sources           # List data sources
POST   /brands/:id/sources           # Connect data source
GET    /brands/:id/sources/:sourceId # Get source details
PUT    /brands/:id/sources/:sourceId # Update source
DELETE /brands/:id/sources/:sourceId # Disconnect source
POST   /brands/:id/sources/:sourceId/sync  # Trigger manual sync
```

### Insight Endpoints

```
GET    /brands/:id/insights          # List insights
GET    /brands/:id/insights/summary  # Get insights summary
GET    /brands/:id/insights/:type    # Get insights by type
POST   /brands/:id/insights/refresh  # Refresh insights
```

**Example Response:**
```json
{
  "insights": {
    "pain_points": [
      {
        "id": "uuid",
        "content": "Shipping takes too long",
        "frequency": 847,
        "sentiment_score": -2.3,
        "impact_score": 89.5,
        "sources": ["reviews", "support_tickets"],
        "examples": ["...", "..."]
      }
    ],
    "benefits": [...],
    "objections": [...],
    "competitor_gaps": [...],
    "viral_hooks": [...]
  },
  "summary": {
    "total_reviews": 12453,
    "avg_sentiment": 3.8,
    "top_pain_point": "Shipping takes too long",
    "top_benefit": "Makes skin glow instantly"
  }
}
```

### Campaign Endpoints

```
GET    /campaigns              # List campaigns
POST   /campaigns              # Create campaign
GET    /campaigns/:id          # Get campaign details
PUT    /campaigns/:id          # Update campaign
DELETE /campaigns/:id          # Delete campaign
POST   /campaigns/:id/launch   # Launch campaign
POST   /campaigns/:id/pause    # Pause campaign
```

### Ad Concept Endpoints

```
POST   /campaigns/:id/concepts/generate  # Generate ad concepts
GET    /campaigns/:id/concepts           # List concepts
GET    /campaigns/:id/concepts/:conceptId # Get concept details
POST   /campaigns/:id/concepts/:conceptId/regenerate # Regenerate
```

**Example Request:**
```json
{
  "campaign_id": "uuid",
  "selected_insights": ["insight_uuid_1", "insight_uuid_2"],
  "count": 10
}
```

**Example Response:**
```json
{
  "concepts": [
    {
      "id": "uuid",
      "hook": "POV: You finally found a protein powder that doesn't taste like chalk",
      "message": "Our customers say it tastes like a milkshake. Try it risk-free.",
      "cta": "Shop Now - Free Shipping",
      "predicted_score": 87,
      "score_breakdown": {
        "relevance": 85,
        "emotional_impact": 92,
        "clarity": 88,
        "differentiation": 84,
        "trend_alignment": 90,
        "trust_factor": 83
      },
      "confidence_interval": {"lower": 82, "upper": 91},
      "reasoning": "Addresses #1 pain point (taste), uses trending POV format, includes risk reversal",
      "predicted_metrics": {
        "ctr": {"lower": 3.2, "upper": 3.8},
        "roas": {"lower": 2.8, "upper": 3.4}
      }
    }
  ]
}
```

### Creative Generation Endpoints

```
POST   /concepts/:id/generate-video   # Generate video variants
POST   /concepts/:id/generate-copy    # Generate copy variants
POST   /concepts/:id/generate-images  # Generate image variants
GET    /ads/:id/preview               # Get ad preview
PUT    /ads/:id/edit                  # Edit ad
```

### Performance Endpoints

```
GET    /campaigns/:id/performance     # Get campaign performance
GET    /ads/:id/performance           # Get ad performance
GET    /performance/dashboard         # Get dashboard data
POST   /performance/export            # Export performance data
```

### Recommendation Endpoints

```
GET    /campaigns/:id/recommendations # Get recommendations
POST   /recommendations/:id/approve   # Approve recommendation
POST   /recommendations/:id/reject    # Reject recommendation
POST   /recommendations/bulk-approve  # Bulk approve
```

### Webhook Endpoints

```
POST   /webhooks/meta                 # Meta Ads webhook
POST   /webhooks/google               # Google Ads webhook
POST   /webhooks/stripe               # Stripe webhook
```

### Rate Limiting

- **Free tier:** 100 requests/hour
- **Starter:** 1,000 requests/hour
- **Growth:** 10,000 requests/hour
- **Scale:** 100,000 requests/hour
- **Enterprise:** Custom

**Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1677721600
```


---

## Infrastructure

### AWS Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USERS / CLIENTS                          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  CloudFront (CDN) + Route 53 (DNS) + WAF (Security)             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  Application Load Balancer (ALB) - Multi-AZ                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    VPC (us-east-1)                               │
├─────────────────────────────────────────────────────────────────┤
│  Public Subnets (AZ-a, AZ-b)                                    │
│  - NAT Gateways                                                  │
│  - Bastion Hosts (optional)                                      │
├─────────────────────────────────────────────────────────────────┤
│  Private Subnets (AZ-a, AZ-b)                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ECS Fargate Cluster                                     │   │
│  │  - API Services (Auto-scaling)                           │   │
│  │  - Background Workers                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  EC2 Auto Scaling Group (GPU instances)                 │   │
│  │  - Video Generation Workers                              │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  Data Subnets (AZ-a, AZ-b)                                      │
│  - RDS PostgreSQL (Multi-AZ)                                    │
│  - ElastiCache Redis (Multi-AZ)                                 │
│  - OpenSearch (Multi-AZ)                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  AWS Services (Managed)                                          │
│  - S3 (Assets, Backups)                                         │
│  - SQS (Message Queues)                                         │
│  - Lambda (Serverless Functions)                                │
│  - Bedrock (AI/ML)                                              │
│  - SageMaker (ML Training/Inference)                            │
│  - Secrets Manager (Credentials)                                │
│  - CloudWatch (Monitoring)                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Compute Resources

**ECS Fargate (API Services):**
- **Service:** API Gateway, User Service, Campaign Service, etc.
- **Task Definition:**
  - CPU: 1 vCPU
  - Memory: 2 GB
  - Container: Python 3.11 + FastAPI
- **Auto-scaling:**
  - Min: 2 tasks per service
  - Max: 20 tasks per service
  - Target: 70% CPU utilization
  - Scale-out: +2 tasks when CPU >70% for 2 minutes
  - Scale-in: -1 task when CPU <30% for 5 minutes

**EC2 (GPU Workers for Video Generation):**
- **Instance Type:** g4dn.xlarge (NVIDIA T4 GPU)
- **AMI:** Deep Learning AMI (Ubuntu)
- **Auto-scaling:**
  - Min: 1 instance
  - Max: 10 instances
  - Target: SQS queue depth (scale when >10 messages)
- **Spot Instances:** 70% spot, 30% on-demand (cost optimization)

**Lambda (Scheduled Jobs):**
- **Functions:**
  - Data sync (hourly)
  - Performance sync (hourly)
  - Model retraining trigger (weekly)
  - Report generation (daily)
- **Configuration:**
  - Runtime: Python 3.11
  - Memory: 512 MB - 3 GB (depending on function)
  - Timeout: 5 minutes (max)
  - Concurrency: 100 (reserved)

### Storage

**S3 Buckets:**

1. **adlily-assets-prod**
   - Purpose: User-uploaded assets (logos, product images)
   - Lifecycle: Intelligent-Tiering
   - Versioning: Enabled
   - Encryption: SSE-S3

2. **adlily-generated-prod**
   - Purpose: Generated videos, images
   - Lifecycle: Standard → Glacier after 90 days
   - Versioning: Disabled
   - Encryption: SSE-S3
   - CloudFront distribution for delivery

3. **adlily-backups-prod**
   - Purpose: Database backups, exports
   - Lifecycle: Standard → Glacier after 30 days
   - Versioning: Enabled
   - Encryption: SSE-KMS

4. **adlily-logs-prod**
   - Purpose: Application logs, audit logs
   - Lifecycle: Standard → Glacier after 90 days
   - Versioning: Disabled
   - Encryption: SSE-S3

**RDS PostgreSQL:**
- **Instance Class:** db.r6g.xlarge (4 vCPU, 32 GB RAM)
- **Storage:** 500 GB gp3 (16,000 IOPS)
- **Multi-AZ:** Enabled (automatic failover)
- **Read Replicas:** 2 (for analytics queries)
- **Backup:**
  - Automated daily snapshots (retained 7 days)
  - Manual snapshots (retained indefinitely)
  - Point-in-time recovery (5 minutes)
- **Encryption:** At rest (KMS), in transit (TLS)

**ElastiCache Redis:**
- **Node Type:** cache.r6g.large (2 vCPU, 13.07 GB RAM)
- **Cluster Mode:** Enabled (3 shards, 2 replicas per shard)
- **Multi-AZ:** Enabled (automatic failover)
- **Backup:** Daily snapshots (retained 7 days)
- **Encryption:** At rest, in transit

**OpenSearch:**
- **Instance Type:** r6g.large.search (2 vCPU, 16 GB RAM)
- **Nodes:** 3 data nodes (Multi-AZ)
- **Storage:** 500 GB EBS (gp3)
- **Dedicated Master:** 3 nodes (for cluster stability)
- **Backup:** Automated snapshots (hourly)

### Networking

**VPC Configuration:**
- **CIDR:** 10.0.0.0/16
- **Subnets:**
  - Public: 10.0.1.0/24 (AZ-a), 10.0.2.0/24 (AZ-b)
  - Private: 10.0.10.0/24 (AZ-a), 10.0.11.0/24 (AZ-b)
  - Data: 10.0.20.0/24 (AZ-a), 10.0.21.0/24 (AZ-b)
- **NAT Gateways:** 2 (one per AZ for high availability)
- **Internet Gateway:** 1

**Security Groups:**

1. **ALB Security Group:**
   - Inbound: 443 (HTTPS) from 0.0.0.0/0
   - Outbound: All to ECS Security Group

2. **ECS Security Group:**
   - Inbound: 8000 (API) from ALB Security Group
   - Outbound: All

3. **RDS Security Group:**
   - Inbound: 5432 (PostgreSQL) from ECS Security Group
   - Outbound: None

4. **Redis Security Group:**
   - Inbound: 6379 from ECS Security Group
   - Outbound: None

**CloudFront Distribution:**
- **Origins:**
  - S3 (generated assets)
  - ALB (API)
- **Cache Behavior:**
  - Assets: Cache for 1 year
  - API: No cache (pass-through)
- **SSL/TLS:** Custom certificate (ACM)
- **Geo-restriction:** None
- **WAF:** Enabled (rate limiting, SQL injection protection)

### Monitoring & Logging

**CloudWatch Metrics:**
- **Application Metrics:**
  - Request count, latency (p50, p95, p99)
  - Error rate (4xx, 5xx)
  - Active users
  - Queue depth (SQS)
  - Generation time (video, image)
- **Infrastructure Metrics:**
  - CPU, memory utilization (ECS, EC2)
  - Database connections, queries/sec (RDS)
  - Cache hit rate (Redis)
  - Disk usage, IOPS (RDS, OpenSearch)

**CloudWatch Alarms:**
- **Critical:**
  - API error rate >5% for 5 minutes → PagerDuty
  - Database CPU >90% for 5 minutes → PagerDuty
  - Any service down → PagerDuty
- **Warning:**
  - API latency p95 >2 seconds for 10 minutes → Email
  - Queue depth >100 for 15 minutes → Email
  - Disk usage >80% → Email

**CloudWatch Logs:**
- **Log Groups:**
  - /aws/ecs/api-service
  - /aws/ecs/worker-service
  - /aws/lambda/data-sync
  - /aws/rds/postgresql
- **Retention:** 30 days (CloudWatch), 90 days (S3)
- **Log Insights:** Enabled (for querying)

**AWS X-Ray:**
- **Tracing:** Enabled for all API requests
- **Sampling:** 10% of requests (to reduce cost)
- **Service Map:** Visualize dependencies
- **Trace Analysis:** Identify bottlenecks

### CI/CD Pipeline

**GitHub Actions Workflow:**

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest tests/
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: adlily-api
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster adlily-prod \
            --service api-service \
            --force-new-deployment
```

**Deployment Strategy:**
- **Blue/Green Deployment:** Zero-downtime deployments
- **Rollback:** Automatic rollback if health checks fail
- **Canary:** 10% traffic to new version, monitor for 10 minutes, then 100%

### Disaster Recovery

**RTO (Recovery Time Objective):** 4 hours  
**RPO (Recovery Point Objective):** 1 hour

**Backup Strategy:**
- **Database:** Automated daily snapshots + continuous backup (point-in-time recovery)
- **S3:** Versioning enabled, cross-region replication (to us-west-2)
- **Configuration:** Infrastructure as Code (Terraform) in Git

**Disaster Recovery Plan:**
1. **Database Failure:**
   - Automatic failover to standby (Multi-AZ) - 2 minutes
   - If complete failure: Restore from snapshot - 30 minutes
2. **Region Failure:**
   - Failover to us-west-2 (manual) - 2 hours
   - DNS update (Route 53) - 5 minutes
   - Data sync from S3 replication - 1 hour
3. **Data Corruption:**
   - Point-in-time recovery (last 5 minutes) - 30 minutes

---

## Security Architecture

### Authentication & Authorization

**Authentication:**
- **Method:** JWT (JSON Web Tokens)
- **Token Expiry:** 1 hour (access token), 30 days (refresh token)
- **Storage:** HttpOnly cookies (web), Secure storage (mobile)
- **MFA:** Optional (TOTP via authenticator app)

**Authorization:**
- **Model:** Role-Based Access Control (RBAC)
- **Roles:**
  - Admin: Full access
  - User: Standard access
  - Viewer: Read-only access
- **Permissions:** Granular (per resource, per action)

**Implementation:**
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> User:
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("sub")
        user = await get_user_by_id(user_id)
        if not user:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

def require_permission(permission: str):
    async def permission_checker(user: User = Depends(get_current_user)):
        if not user.has_permission(permission):
            raise HTTPException(status_code=403, detail="Insufficient permissions")
        return user
    return permission_checker

# Usage
@app.get("/campaigns")
async def list_campaigns(user: User = Depends(require_permission("campaigns:read"))):
    return await get_campaigns(user.id)
```

### Data Security

**Encryption:**
- **At Rest:**
  - Database: AWS RDS encryption (AES-256)
  - S3: SSE-S3 or SSE-KMS
  - EBS volumes: Encrypted
- **In Transit:**
  - TLS 1.3 for all API communication
  - HTTPS only (HTTP redirects to HTTPS)
  - Database connections: SSL/TLS

**PII Protection:**
- **Detection:** Regex patterns, NLP models
- **Redaction:** Automatic removal from reviews/tickets
- **Masking:** Display masked data in UI (e.g., "John D." instead of "John Doe")
- **Audit:** Log all access to PII

**Secrets Management:**
- **AWS Secrets Manager:** API keys, database credentials
- **Rotation:** Automatic rotation every 90 days
- **Access:** IAM roles (no hardcoded credentials)

### API Security

**Rate Limiting:**
- **Implementation:** Redis-based (token bucket algorithm)
- **Limits:** Per user, per IP, per endpoint
- **Response:** 429 Too Many Requests (with Retry-After header)

**Input Validation:**
- **Pydantic Models:** Type checking, validation
- **Sanitization:** Remove HTML, SQL injection attempts
- **Size Limits:** Max request body 10 MB

**CORS:**
- **Allowed Origins:** app.adlily.com, localhost (dev)
- **Allowed Methods:** GET, POST, PUT, DELETE
- **Allowed Headers:** Authorization, Content-Type
- **Credentials:** Allowed

**CSRF Protection:**
- **Method:** Double-submit cookie pattern
- **Token:** Generated per session, validated on state-changing requests

### Infrastructure Security

**Network Security:**
- **VPC:** Isolated network
- **Security Groups:** Least privilege (only necessary ports)
- **NACLs:** Additional layer of defense
- **Private Subnets:** No direct internet access (via NAT)

**WAF (Web Application Firewall):**
- **Rules:**
  - Rate limiting (1000 req/5min per IP)
  - SQL injection protection
  - XSS protection
  - Known bad IPs (blocklist)
- **Logging:** All blocked requests logged

**DDoS Protection:**
- **AWS Shield Standard:** Automatic (free)
- **AWS Shield Advanced:** Optional (for enterprise)
- **CloudFront:** Absorbs traffic spikes

**Vulnerability Management:**
- **Dependency Scanning:** Dependabot (GitHub)
- **Container Scanning:** AWS ECR image scanning
- **Penetration Testing:** Annual (third-party)
- **Bug Bounty:** HackerOne (future)

### Compliance

**SOC 2 Type II:**
- **Controls:** Access control, encryption, monitoring, incident response
- **Audit:** Annual (third-party auditor)
- **Status:** In progress (target: Q3 2026)

**GDPR:**
- **Data Subject Rights:** Access, rectification, erasure, portability
- **Consent:** Explicit consent for data processing
- **Data Processing Agreement:** With customers
- **Data Retention:** Configurable (default: 2 years)

**CCPA:**
- **Consumer Rights:** Know, delete, opt-out
- **Privacy Policy:** Transparent data practices
- **Do Not Sell:** Honored (we don't sell data)

### Incident Response

**Incident Response Plan:**

1. **Detection:**
   - Automated alerts (CloudWatch, X-Ray)
   - User reports
   - Security monitoring

2. **Triage:**
   - Assess severity (P0-P4)
   - Assign incident commander
   - Create incident channel (Slack)

3. **Containment:**
   - Isolate affected systems
   - Block malicious traffic
   - Preserve evidence

4. **Eradication:**
   - Remove threat
   - Patch vulnerabilities
   - Update security rules

5. **Recovery:**
   - Restore from backups (if needed)
   - Verify system integrity
   - Resume normal operations

6. **Post-Mortem:**
   - Document incident
   - Identify root cause
   - Implement preventive measures
   - Update runbooks

**Incident Severity:**
- **P0 (Critical):** Data breach, complete outage - Response: Immediate
- **P1 (High):** Partial outage, security vulnerability - Response: <1 hour
- **P2 (Medium):** Performance degradation - Response: <4 hours
- **P3 (Low):** Minor issues - Response: <24 hours
- **P4 (Informational):** No impact - Response: Next sprint

---

## Scalability & Performance

### Performance Targets

| Metric | Target | Current (MVP) |
|--------|--------|---------------|
| API Response Time (p95) | <500ms | <800ms |
| Dashboard Load Time | <2s | <3s |
| Video Generation Time | <3 min | <5 min |
| Insight Processing (10K reviews) | <5 min | <10 min |
| Uptime | 99.9% | 99.5% |
| Concurrent Users | 10,000+ | 1,000 |

### Scalability Strategy

**Horizontal Scaling:**
- **API Services:** Auto-scaling ECS tasks (2-20 per service)
- **Workers:** Auto-scaling EC2 instances (1-10 for GPU)
- **Database:** Read replicas (2-5 for read-heavy queries)

**Vertical Scaling:**
- **Database:** Upgrade instance class (r6g.xlarge → r6g.2xlarge)
- **Cache:** Upgrade node type (r6g.large → r6g.xlarge)

**Caching Strategy:**

**1. Application-Level Cache (Redis):**
- **User sessions:** 1 hour TTL
- **API responses:** 5 minutes TTL (for expensive queries)
- **Insights summary:** 1 hour TTL
- **Ad scores:** 10 minutes TTL

**2. Database Query Cache:**
- **PostgreSQL:** Shared buffers (8 GB), effective cache size (24 GB)
- **Read replicas:** For analytics queries (offload primary)

**3. CDN Cache (CloudFront):**
- **Static assets:** 1 year TTL
- **Generated videos:** 30 days TTL
- **API responses:** No cache (pass-through)

**Database Optimization:**

**1. Indexing:**
- All foreign keys indexed
- Composite indexes for common queries
- Partial indexes for filtered queries

**2. Partitioning:**
- Time-series tables partitioned by month
- Automatic partition creation (cron job)

**3. Query Optimization:**
- Use EXPLAIN ANALYZE for slow queries
- Optimize N+1 queries (use joins or batch loading)
- Materialized views for complex aggregations

**4. Connection Pooling:**
- PgBouncer (transaction pooling)
- Max connections: 100 (per service)

**Async Processing:**
- **SQS Queues:**
  - data-ingestion-queue (FIFO)
  - video-generation-queue (Standard)
  - performance-sync-queue (Standard)
- **Dead Letter Queues:** For failed messages (retry 3 times)
- **Visibility Timeout:** 5 minutes (video gen), 1 minute (others)

### Load Testing

**Tools:**
- **Locust:** API load testing
- **k6:** Performance testing
- **AWS Load Testing:** Distributed load testing

**Test Scenarios:**
1. **Normal Load:** 100 concurrent users, 1000 req/min
2. **Peak Load:** 500 concurrent users, 5000 req/min
3. **Stress Test:** 1000 concurrent users, 10000 req/min
4. **Spike Test:** 0 → 1000 users in 1 minute

**Performance Benchmarks:**
- **API Endpoints:**
  - GET /campaigns: <100ms (p95)
  - POST /campaigns: <500ms (p95)
  - POST /concepts/generate: <30s (p95)
  - POST /generate-video: <3min (p95)

---

## Appendix

### Technology Decisions

**Why FastAPI?**
- High performance (async/await)
- Automatic API documentation (OpenAPI)
- Type safety (Pydantic)
- Easy to learn and use
- Great ecosystem

**Why PostgreSQL?**
- ACID compliance (data integrity)
- JSONB support (flexible schema)
- Full-text search (built-in)
- Mature, battle-tested
- Great AWS support (RDS)

**Why AWS Bedrock?**
- Managed service (no infrastructure)
- Multiple models (Claude, Stable Diffusion, Titan)
- Pay-per-use (cost-effective)
- Low latency (AWS network)
- Enterprise-ready (security, compliance)

**Why ECS Fargate?**
- Serverless containers (no EC2 management)
- Auto-scaling (built-in)
- Cost-effective (pay for what you use)
- Easy deployment (Docker)
- Integrates with AWS ecosystem

### Glossary

- **ECS:** Elastic Container Service (AWS)
- **Fargate:** Serverless compute for containers
- **RDS:** Relational Database Service (AWS)
- **SQS:** Simple Queue Service (AWS)
- **ALB:** Application Load Balancer
- **VPC:** Virtual Private Cloud
- **IAM:** Identity and Access Management
- **KMS:** Key Management Service
- **ACM:** AWS Certificate Manager
- **WAF:** Web Application Firewall

---

**Document Owner:** Engineering Team  
**Contributors:** DevOps, Security, Product  
**Last Updated:** February 15, 2026  
**Next Review:** March 15, 2026

