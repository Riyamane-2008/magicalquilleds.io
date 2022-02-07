using CognizantReflect.Api.Adapters.Interfaces;
using CognizantReflect.Api.Helpers;
using CognizantReflect.Api.Helpers.Interfaces;
using CognizantReflect.Api.Models;
using CognizantReflect.Api.Models.MakingTimeForMeQuiz;
using Microsoft.Extensions.Options;
using MongoDB.Bson;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using CognizantReflect.Api.Models.LearningMythsQuiz;
using MongoDB.Driver;

namespace CognizantReflect.Api.Adapters
{
    internal class MakingTimeForMeQuizAdapter : IMakingTimeForMeQuizAdapter
    {
        private readonly IMongoClientHelper<MakingTimeForMeQuiz> _makingTimeForMeQuiz;
        private readonly IMongoClientHelper<MakingTimeForMeQuizAttempts> _makingTimeForMeQuizAttempts;
        private readonly string _makingTimeForMeQuizCollection;
        private readonly string _makingTimeForMeQuizAttemptsCollection;
        public MakingTimeForMeQuizAdapter(IMongoClientHelper<MakingTimeForMeQuiz> makingTimeForMeQuiz, IMongoClientHelper<MakingTimeForMeQuizAttempts> makingTimeForMeQuizAttempts, IOptions<MongoDbSettings> options)
        {
            _makingTimeForMeQuiz = makingTimeForMeQuiz;
            _makingTimeForMeQuizAttempts = makingTimeForMeQuizAttempts;
            _makingTimeForMeQuizCollection = options.Value.MakingTimeForMeQuizCollection;
            _makingTimeForMeQuizAttemptsCollection = options.Value.MakingTimeForMeQuizAttemptsCollection;
        }
        public List<MakingTimeForMeQuiz> GetMakingTimeForMeQuizzes()
        {
            var result = _makingTimeForMeQuiz.GetTotalRecords(_makingTimeForMeQuizCollection);
            List<MakingTimeForMeQuiz> makingTimeForMeQuizzes = new List<MakingTimeForMeQuiz>();

            foreach (var item in result)
            {
                item.TryGetValue("_id", out BsonValue idValue);
                item.TryGetValue("question", out BsonValue questionValue);
                item.TryGetValue("scores", out BsonValue scoreValue);
                item.TryGetValue("updatetimestamp", out BsonValue updatetimestampValue);
                makingTimeForMeQuizzes.Add(new MakingTimeForMeQuiz
                {
                    id = Common.IntegerNullCheck(idValue),
                    question = Common.StringNullCheck(questionValue),
                    scores = new Score { always = Common.IntegerNullCheck(scoreValue?[0]), never = Common.IntegerNullCheck(scoreValue?[1]),
                        often = Common.IntegerNullCheck(scoreValue?[2]), rarely = Common.IntegerNullCheck(scoreValue?[3]), sometimes = Common.IntegerNullCheck(scoreValue?[4]) },
                    updatetimestamp = Common.StringNullCheck(updatetimestampValue)
                }); ;
            }
            return makingTimeForMeQuizzes;
        }

        public int InsertMakingTimeForMeQuiz(MakingTimeForMeQuiz makingTimeForMeQuiz)
        {
            _makingTimeForMeQuiz.InsertOne(makingTimeForMeQuiz, _makingTimeForMeQuizCollection);
            return 1;
        }

        public int InsertMakingTimeForMeQuizAttempts(List<MakingTimeForMeQuizAttempts> makingTimeForMeQuizAttempts)
        {
            _makingTimeForMeQuizAttempts.InsertAll(makingTimeForMeQuizAttempts,_makingTimeForMeQuizAttemptsCollection);
            return 1;
        }

        public MakingTimeForMeQuizAttempts GetLatestId()
        {
            var sort = Builders<MakingTimeForMeQuizAttempts>.Sort.Descending(f => f.id);
            return _makingTimeForMeQuizAttempts.GetLatestId(_makingTimeForMeQuizAttemptsCollection, sort);
        }

        public MakingTimeForMeQuizAttempts GetLatestAttemptByUser(string userid)
        {
            var filter = Builders<MakingTimeForMeQuizAttempts>.Filter.Eq("userid", userid);
            return _makingTimeForMeQuizAttempts.GetData(filter, _makingTimeForMeQuizAttemptsCollection)?
                .OrderByDescending(x => x.attemptcount)?.FirstOrDefault();
        }
    }
}
